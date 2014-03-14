Original Class
--------------
This class is a slighly adapted class from a real-world project. It performs 
validation of an image according to criteria which are passed as a parameter::

    class ImageValidator
    {
        public function validate($value, Constraint $constraint)
        {
            if (null === $value || '' === $value) {
                return;
            }
    
            if (null === $constraint->minWidth && null === $constraint->maxWidth) {
                return;
            }
    
            $size = @getimagesize($value);
            if (empty($size) || ($size[0] === 0) || ($size[1] === 0)) {
                $this->context->addViolation($constraint->sizeNotDetectedMessage);
    
                return;
            }
    
            $width  = $size[0];
            $height = $size[1];
    
            if ($constraint->minWidth) {
                if (!ctype_digit((string) $constraint->minWidth)) {
                    throw new ConstraintDefinitionException(sprintf('"%s" is not a valid minimum width', $constraint->minWidth));
                }
    
                if ($width < $constraint->minWidth) {
                    $this->context->addViolation($constraint->minWidthMessage, array(
                        '{{ width }}'    => $width,
                        '{{ min_width }}' => $constraint->minWidth
                    ));
    
                    return;
                }
            }
    
            if ($constraint->maxWidth) {
                if (!ctype_digit((string) $constraint->maxWidth)) {
                    throw new ConstraintDefinitionException(sprintf('"%s" is not a valid maximum width', $constraint->maxWidth));
                }
    
                if ($width > $constraint->maxWidth) {
                    $this->context->addViolation($constraint->maxWidthMessage, array(
                        '{{ width }}'    => $width,
                        '{{ max_width }}' => $constraint->maxWidth
                    ));
    
                    return;
                }
            }
        }
    }

Creating a Composing Method
---------------------------
The current ``validate()`` method can be rawly composed into three different 
steps:

1. Validating the input value's type
2. Validating the constraint
3. Validating the image's size

.. code-block :: php

    class ImageValidator
    {
        public function validate($value, Constraint $constraint)
        {
            if ( ! $this->isInputTypeValid($value)) {
                return;
            }
            
            if ( ! $this->isConstraintValid($constraint)) {
                return;
            }

            $this->validateImageSize($value, $constraint);
        }

        private function isInputValid($value)
        {
            return $value !== null && $value !== '';
        }

        private function isConstraintValid(Constraint $constraint)
        {
            return $constraint->minWidth !== null || $constraint->maxWidth !== null;
        }

        private function validateImageSize($value, Constraint $constraint)
        {
            $size = @getimagesize($value);
            if (empty($size) || ($size[0] === 0) || ($size[1] === 0)) {
                $this->context->addViolation($constraint->sizeNotDetectedMessage);
    
                return;
            }
    
            // ...
        }
    }

The ``validateImageSize`` is still quite big. We could go ahead an re-apply the
**Compose Method** refactoring to it as well to split it up further.