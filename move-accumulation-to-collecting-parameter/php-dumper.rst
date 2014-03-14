The following code is a slightly adapted version of a real-world class. We will
now refactor it to use the **Collecting Parameter** pattern.

Step 1: Identify Accumulation Method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The accumulation method in our example is ``addService`` which accumulates its
results in the local ``$return`` variable::

    class PhpDumper
    {
        // ...

        private function addService($id, $definition)
        {
            $return = array();

            $return[] = '/**';
            if ($definition->isSynthetic()) {
                $return[] = ' * @throws RuntimeException always since this service is expected to be injected dynamically';
            } elseif ($class = $definition->getClass()) {
                $return[] = sprintf(" * @return %s A %s instance.", 0 === strpos($class, '%') ? 'object' : $class, $class);
            } elseif ($definition->getFactoryClass()) {
                $return[] = sprintf(' * @return object An instance returned by %s::%s().', $definition->getFactoryClass(), $definition->getFactoryMethod());
            } elseif ($definition->getFactoryService()) {
                $return[] = sprintf(' * @return object An instance returned by %s::%s().', $definition->getFactoryService(), $definition->getFactoryMethod());
            }

            $return[] = " */";

            $return[] = "public function get{$this->camelize($id)}Service()";
            $return[] = "{";

            $return[] = $this->addServiceBody($id, $definition);
            $return[] = "}";

            return implode("\n", $return);
        }

        // ...
    }

Step 2a: Extract First Operation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. note ::
    In order to apply the **Collecting Parameter pattern** in PHP, we either need to define the 
    parameter as passed-by-reference, or we need to pass an object. In this first
    example, we will preserve the array. In the second example (step 2b see below), 
    we will demonstrate the refactoring using a new object.

The first step in the ``addService`` method is to add a doc-comment for the current
service. We will extract the related code to a new method and pass the ``$return`` 
variable as a parameter to it::

    class PhpDumper
    {
        // ...

        private function addService($id, $definition)
        {
            $return = array();
            $this->addServiceDocComment($id, $definition, $return);

            $return[] = "public function get{$this->camelize($id)}Service()";
            $return[] = "{";

            $return[] = $this->addServiceBody($id, $definition);
            $return[] = "}";

            return implode("\n", $return);
        }

        private function addServiceDocComment($id, $definition, array &$return)
        {
            $return[] = '/**';
            if ($definition->isSynthetic()) {
                $return[] = ' * @throws RuntimeException always since this service is expected to be injected dynamically';
            } elseif ($class = $definition->getClass()) {
                $return[] = sprintf(" * @return %s A %s instance.", 0 === strpos($class, '%') ? 'object' : $class, $class);
            } elseif ($definition->getFactoryClass()) {
                $return[] = sprintf(' * @return object An instance returned by %s::%s().', $definition->getFactoryClass(), $definition->getFactoryMethod());
            } elseif ($definition->getFactoryService()) {
                $return[] = sprintf(' * @return object An instance returned by %s::%s().', $definition->getFactoryService(), $definition->getFactoryMethod());
            }

            $return[] = " */";
        }

        // ...
    }

Step 3: Repeating Step 2
~~~~~~~~~~~~~~~~~~~~~~~~~
After applying step 2 to the remaining body of the ``addService`` method, our
refactored code looks like this::

    class PhpDumper
    {
        // ...

        private function addService($id, $definition)
        {
            $return = array();
            $this->addServiceDocComment($id, $definition, $return);
            $this->addServiceCode($id, $definition, $return);

            return implode("\n", $return);
        }

        private function addServiceCode($id, $definition, array &$return)
        {
            $return[] = "public function get{$this->camelize($id)}Service()";
            $return[] = "{";

            $return[] = $this->addServiceBody($id, $definition);
            $return[] = "}";
        }

        private function addServiceDocComment($id, $definition, array &$return)
        {
            $return[] = '/**';
            if ($definition->isSynthetic()) {
                $return[] = ' * @throws RuntimeException always since this service is expected to be injected dynamically';
            } elseif ($class = $definition->getClass()) {
                $return[] = sprintf(" * @return %s A %s instance.", 0 === strpos($class, '%') ? 'object' : $class, $class);
            } elseif ($definition->getFactoryClass()) {
                $return[] = sprintf(' * @return object An instance returned by %s::%s().', $definition->getFactoryClass(), $definition->getFactoryMethod());
            } elseif ($definition->getFactoryService()) {
                $return[] = sprintf(' * @return object An instance returned by %s::%s().', $definition->getFactoryService(), $definition->getFactoryMethod());
            }

            $return[] = " */";
        }

        // ...
    }

We could now also go ahead and apply this refactoring to the newly created methods
to further decrease their size if desired.

--------------------------------------

Alternative Step 2b: Extract First Operation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
In this chapter, we will perform the same refactoring as in step 2a except that
we will be using an object as **Collecting Parameter**. We will call this new object 
``StringBuilder``::

    class StringBuilder
    {
        private $content = '';

        public function appendln($content)
        {
            $this->content .= $content."\n";

            return $this;
        }

        public function getContent()
        {
            return $this->content;
        }
    }

    class PhpDumper
    {
        // ...

        private function addService($id, $definition)
        {
            $sb = new StringBuilder();
            $this->addServiceDocComment($id, $definition, $sb);

            $sb
                ->appendln("public function get{$this->camelize($id)}Service()")
                ->appendln("{")
            ;

            $this->addServiceBody($id, $definition, $sb);

            $sb->appendln("}");

            return $sb->getContent();
        }

        private function addServiceDocComment($id, $definition, StringBuilder $sb)
        {
            $sb->appendln('/**');
            if ($definition->isSynthetic()) {
                $sb->appendln(' * @throws RuntimeException always since this service is expected to be injected dynamically');
            } elseif ($class = $definition->getClass()) {
                $sb->appendln(sprintf(" * @return %s A %s instance.", 0 === strpos($class, '%') ? 'object' : $class, $class));
            } elseif ($definition->getFactoryClass()) {
                $sb->appendln(sprintf(' * @return object An instance returned by %s::%s().', $definition->getFactoryClass(), $definition->getFactoryMethod()));
            } elseif ($definition->getFactoryService()) {
                $sb->appendln(sprintf(' * @return object An instance returned by %s::%s().', $definition->getFactoryService(), $definition->getFactoryMethod()));
            }

            $sb->appendln(" */");
        }

        // ...
    }
    
In contrast to step 2a, using an object gives us a bit more flexibility. We could for
example start to add convenience methods to it. In our case, ``appendDocComment``,
``appendDocCommentStart`` or indentation handling would be good candidates for 
such methods, and would help avoid duplication.