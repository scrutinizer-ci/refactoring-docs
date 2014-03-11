Step 1: Determine what to extract
---------------------------------

Let's assume the following class::

    class BlogPost
    {
        // ...
        private $previewTextRst;
        private $previewTextHtml;
        private $textRst;
        private $textHtml;

        // ...

        public function setPreviewText($rst, $html)
        {
            $this->previewTextRst = $rst;
            $this->previewTextHtml = $html;
        }

        public function setText($rst, $html)
        {
            $this->textRst = $rst;
            $this->textHtml = $html;
        }

        // ...
    }

As you can see, we have two sets of fields which are grouped together and some
operations which only perform work on a specific subset of fields. In this example,
it is very obvious since we also have some duplication which we can reduce by
extracting a class.

Step 2: Extracting Class
------------------------
We will continue by introducing a new ``Text`` class and move the ``textRst``
and ``textHtml`` fields to it::

    class Text
    {
        private $rst;
        private $html;

        public function __construct($rst, $html)
        {
            $this->rst = $rst;
            $this->html = $html;
        }

        public function getRst()
        {
            return $this->rst;
        }

        public function getHtml()
        {
            return $this->html;
        }
    }

    class BlogPost
    {
        private $previewText;
        private $text;
        
        public function setPreviewText($rst, $html)
        {
            $this->previewText = new Text($rst, $html);
        }

        public function setText($rst, $html)
        {
            $this->text = new Text($rst, $html);
        }
    }

Step 3: Rename old class
------------------------
In our case, the name of the old class is still fitting, we do not need to 
rename it. So, we will just skip this step.

Step 4: Review Interfaces and Links of each class
-------------------------------------------------
Our classes only have a unidirectional link, the ``Text`` class does not know
where it is being used, and it also does not need to know. This also allows us
to use the ``Text`` class in classes other than ``BlogPost``. So, there is 
nothing to change here as well.

Step 5: Value Object vs. Reference Object
-----------------------------------------
In general, prefer to use a value object as this limits the scope of how and
when the object can be modified. In our case, we decided to use a value object:
If the text is changed, a new object must be created. This way we ensure that 
any modification must go through the owning class, in our case ``BlogPost``.