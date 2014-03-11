Background
----------

As example, we will refactor a Worker class which subscribes to several AMQP queues
and processes incoming messages. This class was taken from a real-world open-source
project and was slightly adapted for the purposes of this example::

    class WorkflowWorker
    {
        private $entityManager;
        private $serializer;
    
        public function __construct(EntityManager $entityManager, Serializer $serializer, Channel $channel)
        { 
            $this->entityManager = $entityManager;
            $this->serializer = $serializer;
            
            $channel->subscribe(array($this, 'onMessage'));
        }

        public function onMessage($queue, AMQPMessage $message)
        {
            switch ($queue) {
                case 'workflow_execution_termination':
                    return $this->consumeExecutionTermination($message);

                case 'workflow_execution_details':
                    return $this->consumeExecutionDetails($message);

                default:
                    throw new \LogicException('Unsupported queue.');
            }
        }
    
        public function consumeExecutionDetails(AMQPMessage $message)
        {
            $input = json_decode($message->body, true);
    
            if ( ! isset($input['execution_id'])) {
                throw new \InvalidArgumentException('"execution_id" attribute was not set.');
            }
    
            $execution = $this->entityManager->find('Workflow:WorkflowExecution', $input['execution_id']);
            if (null === $execution) {
                throw new \InvalidArgumentException(
                    sprintf('There is no execution with id "%s".', $input['execution_id']));
            }
    
            return $this->serializer->serialize($execution, 'json');
        }
    
        public function consumeExecutionTermination(AMQPMessage $message)
        {
            $input = json_decode($message->body, true);
    
            /** @var $execution WorkflowExecution */
            $execution = $this->entityManager->getRepository('Workflow:WorkflowExecution')
                                                ->getByIdExclusive($input['execution_id']);
    
            $execution->terminate();
            $this->entityManager->persist($execution);
            $this->entityManager->flush();
    
            return '';
        }
    
        // ...
    }

Step 1: Locate the Request Handling Code 
----------------------------------------
This step is quite easy as the handling code was already extracted into separate
methods. Basically, both callback methods (``consumeExecutionDetails`` and
``consumeExecutionTermination``) qualify as request-handling code and will 
be moved to command classes.

Step 2: Repeat Step 1
---------------------
We can safely skip this as all methods are already extracted.

Step 3: Extract Request-Handling Methods 
----------------------------------------
Finally, it's time to write some code, or more precisely move code around. We 
will extract all the request-handling methods to their own classes. We will not
yet change any code though.

.. tip ::
    If commands call other methods on the class containing the conditional dispatcher,
    i.e. ``WorkflowWorker``, we can temporarily make these methods public and inject 
    the worker as a dependency into the respective commands.

.. code-block :: php

    class ConsumeExecutionDetailsCommand
    {
        private $entityManager;
        private $serializer;
    
        public function __construct(EntityManager $entityManager, Serializer $serializer)
        {
            $this->entityManager = $entityManager;
            $this->serializer = $serializer;
        }
    
        public function execute(AMQPMessage $message)
        {
            $input = json_decode($message->body, true);
    
            if ( ! isset($input['execution_id'])) {
                throw new \InvalidArgumentException('"execution_id" attribute was not set.');
            }
    
            $execution = $this->entityManager->find('Workflow:WorkflowExecution', $input['execution_id']);
            if (null === $execution) {
                throw new \InvalidArgumentException(
                    sprintf('There is no execution with id "%s".', $input['execution_id']));
            }
    
            return $this->serializer->serialize($execution, 'json');
        }
    }
    
    class ConsumeExecutionTerminationCommand
    {
        private $entityManager;
    
        public function __construct(EntityManager $entityManager)
        {
            $this->entityManager = $entityManager;
        }
    
        public function execute(AMQPMessage $message)
        {
            $input = json_decode($message->body, true);
    
            /** @var $execution WorkflowExecution */
            $execution = $this->entityManager->getRepository('Workflow:WorkflowExecution')
                                                ->getByIdExclusive($input['execution_id']);
    
            $execution->terminate();
            $this->entityManager->persist($execution);
            $this->entityManager->flush();
    
            return '';
        }
    }
    
    class WorkflowWorker
    {
        private $executionDetailsCommand;
        private $executionTerminationCommand;
    
        public function __construct(EntityManager $entityManager, Serializer $serializer, Channel $channel)
        {
            $this->executionDetailsCommand = new ConsumeExecutionDetailsCommand($entityManager, $serializer);
            $this->executionTerminationCommand = new ConsumeExecutionTerminationCommand($entityManager);
            
            $channel->subscribe(array($this, 'onMessage'));
        }

        public function onMessage($queue, AMQPMessage $message)
        {
            switch ($queue) {
                case 'workflow_execution_termination':
                    return $this->consumeExecutionTermination($message);

                case 'workflow_execution_details':
                    return $this->consumeExecutionDetails($message);

                default:
                    throw new \LogicException('Unsupported queue.');
            }
        }
    
        public function consumeExecutionDetails(AMQPMessage $message)
        {
            return $this->executionDetailsCommand->execute($message);
        }
    
        public function consumeExecutionTermination(AMQPMessage $message)
        {
            return $this->executionTerminationCommand->execute($message);
        }
    
        // ...
    }
    
The request-handling logic is now in their own classes. The code is not really
desirable yet as we have some duplication, but the important part is that to 
all outside classes nothing has changed. You can also still run your tests and
they should still pass. In practice, you can of course skip steps and arrange
the code in its final form directly without performing a step-by-step transformation.

Step 4 & 5: Define a Command base class/interface and extend/implement it
-------------------------------------------------------------------------
This step requires some analysis of the extracted commands. If we take a look at
both, we see that both commands have a dependency on the ``EntityManager`` this
is a first candidate for placing it in a base command. While the ``Serializer`` 
is only a dependency for one command, we will still pull it up into the base class
as it seems general enough to be useful for future commands.

The choice for the signature of the ``execute`` method is quite easy as it is 
already the same for both classes. There is no need to change it.

Our final command classes looks like this::

    abstract class Command
    {
        private $entityManager;
        private $serializer;
    
        public function __construct(EntityManager $entityManager, Serializer $serializer)
        {
            $this->entityManager = $entityManager;
            $this->serializer = $serializer;
        }
    
        abstract public function execute(AMQPMessage $message);
    }
    
    class ConsumeExecutionDetailsCommand extends Command
    {
        public function execute(AMQPMessage $message)
        {
            // ...
        }
    }
    
    class ConsumeExecutionTerminationCommand extends Command
    {
        public function execute(AMQPMessage $message)
        {
            // ...
        }
    }

Step 6 & 7: Define Command Map and Replace Conditional Dispatcher
-----------------------------------------------------------------
Now, the last thing left to do is to clean up the ``WorkflowWorker``. We will
remove the fields for the different commands and replace them with an array.
Besides we will also get rid of the switch statement for dispatching the request::

    class WorkflowWorker
    {
        private $commands = array();
    
        public function __construct(EntityManager $entityManager, Serializer $serializer, Channel $channel)
        {
            $this->commands = array(
                'workflow_execution_termination' => 
                    new ConsumeExecutionTerminationCommand($entityManager, $serializer),
                'workflow_execution_details' => 
                    new ConsumeExecutionDetailsCommand($entityManager, $serializer),
            );
    
            $channel->subscribe(array($this, 'onMessage'));
        }
    
        public function onMessage($queue, AMQPMessage $message)
        {
            if ( ! isset($this->commands[$queue])) {
                throw new \LogicException('Unsupported queue.');
            }
            
            return $this->commands[$queue]->execute($message);
        }
    
        // ...
    }

That's it! We successfully refactored the ``WorkflowWorker`` to the command
pattern.