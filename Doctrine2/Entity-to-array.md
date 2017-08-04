## Doctrine2 export entity to array
```code
<?php

trait EntitySerializerTrait
{
    protected $_em;

    public function setEntityManager($em)
    {
        $this->_em = $em;

        return $this;
    }

    public function serialize($entityObjects)
    {
        $entityFields = [];

        if (is_array($entityObjects)) {
            $entityFields = array_map(function ($entityObject) {
                return call_user_func([$this, 'serializeEntity'], $entityObject);
            }, $entityObjects);
        } else {
            $entityFields = $this->serializeEntity($entityObjects);
        }

        return $entityFields;
    }

    protected function serializeEntity($entityObject)
    {
        $data = [];

        $className = get_class($entityObject);
        $metaData = $this->_em->getClassMetadata($className);

        foreach ($metaData->fieldMappings as $field => $mapping) {
            $method = 'get'.$this->convertHump($field);
            $data[$field] = call_user_func(array($entityObject, $method));
        }

        foreach ($metaData->associationMappings as $field => $mapping) {
            // Sort of entity object
            $object = $metaData->reflFields[$field]->getValue($entityObject);

            $data[$field] = $this->serialize($object);
        }

        return $data;
    }

    protected function convertHump($str, $ucfirst = true)
    {
        $str = str_replace(' ', '', ucwords(str_replace('_', ' ', $str)));

        return $ucfirst ? $str : lcfirst($str);
    }
}
```
