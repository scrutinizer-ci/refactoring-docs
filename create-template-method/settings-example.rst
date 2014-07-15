Original classes
----------------
We have following classes, with similar methods for loading settings from database::

    class FrontendController
    {
        public function getFrontendSettings()
        {
            $dbSettings = $this->loadSettingsFromDb('frontend');
            
            $settings = array();
            foreach($dbSettings as $s) {
                $settings[$s['section'][$s['key']] = $s['value'];
            }
            
            return $settings;
        }
    }
    
    class AdminController
    {
        public function getAdminSettings()
        {
            $dbSettings = $this->loadSettingsFromDb('admin');
            
            $settings = array();
            foreach($dbSettings as $s) {
                $settings[$s['section'][$s['key']] = $s['value'];
            }
            
            return $settings;
        }
    }


1) Step 1 - Find two methods in sub-classes which perform similar steps 
---------
::

    class FrontendController
    {
        protected function getNamespace()
        {
            return 'frontend';
        }
    
        public function getFrontendSettings()
        {
            $dbSettings = $this->loadSettingsFromDb($this->getNamespace());
            
            ...
        }
    }
    
    class AdminController
    {
        protected function getNamespace()
        {
            return 'admin';
        }
    
        public function getAdminSettings()
        {
            $dbSettings = $this->loadSettingsFromDb($this->getNamespace());
            
            ...
        }
    }    



Step 2 - Unify the names of the now identical methods 
------
::

    class FrontendController
    {
        public function getSettings()
        {
            ...
        }
        
        ...
    }
    
    class AdminController
    {
        public function getSettings()
        {
            ...
        }
        
        ...
    }

Step 3 - Consolidate method in the parent class 
------
::

    class BaseController
    {
        public function getSettings()
        {
            ...
        }
    }

    class FrontendController extends BaseController
    {
        protected function getNamespace()
        {
            return 'frontend';
        }
    }
    
    class AdminController extends BaseController
    {
        protected function getNamespace()
        {
            return 'admin';
        }
    }
    
Step 4 - Create abstract methods in the parent class 
------
::

    class BaseController
    {
        public function getSettings()
        {
            $dbSettings = $this->loadSettingsFromDb($this->getNamespace());
            
            $settings = array();
            foreach($dbSettings as $s) {
                $settings[$s['section'][$s['key']] = $s['value'];
            }
            
            return $settings;
        }
        
        abstract protected function getNamespace();
    }
