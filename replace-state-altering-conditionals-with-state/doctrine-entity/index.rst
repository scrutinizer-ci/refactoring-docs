In this example, we will take a look at how we can refactor a Doctrine entity to
the `State pattern <http://en.wikipedia.org/wiki/State_pattern>`_. 

Let's assume that we start with the following code::

    use Doctrine\ORM\Mapping as ORM;

    class Job
    {
        const STATE_NEW = 'new';
        const STATE_PENDING = 'pending';
        const STATE_CANCELED = 'canceled';
        const STATE_RUNNING = 'running';
        const STATE_FINISHED = 'finished';
        const STATE_FAILED = 'failed';
        const STATE_TERMINATED = 'terminated';

        /** @ORM\Column(type = "datetime", nullable = true) */
        private $closedAt;

        /** @ORM\Column(type = "string") */
        private $state = self::STATE_NEW;

        public function getState()
        {
            return $this->state;
        }

        public function isFinalState()
        {
            $finalStates =  array(
                self::STATE_CANCELED, 
                self::STATE_FAILED,
                self::STATE_TERMINATED, 
                self::STATE_FINISHED
            );

            return in_array($this->state, $finalStates, true);
        }

        public function getClosedAt()
        {
            if ( ! $this->isFinalState()) {
                throw new \RuntimeException('The Job is not in the final state, yet.');
            }

            return $this->closedAt;
        }
    }