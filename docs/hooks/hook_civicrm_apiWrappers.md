# hook_civicrm_apiWrappers

## Description

This hook allows you to add (or override/remove: use caution!) methods to be called before and after api calls to modify either the parameters or the result of the call.

The methods must be implemented on an `API_Wrapper` object which is added to the array passed by-reference. See below for more information on the `API_Wrapper` class.

Introduced in CiviCRM 4.4.0.

## Definition

```php
    /**
     * Implements hook_civicrm_apiWrappers
     */
    function myextension_civicrm_apiWrappers(&$wrappers, $apiRequest) {
      //&apiWrappers is an array of wrappers, you can add your(s) with the hook.
      // You can use the apiRequest to decide if you want to add the wrapper (eg. only wrap api.Contact.create)
      if ($apiRequest['entity'] == 'Contact' && $apiRequest['action'] == 'create') {
        $wrappers[] = new CRM_Myextension_APIWrapper();
      }
    }
```

## Wrapper class

The wrapper is an object that contains two methods:

 * `fromApiInput()` allows for modification of the params before doing the api call.

 * `toApiInput()` allows for modification of the result of the call

These methods will be called for every API call unless the `hook_civicrm_apiWrappers()` implementation conditionally registers the object. One way to optimize this is to check for the API Entity in `hook_civicrm_apiWrappers()` and to check for the API action in the wrapper methods.

To take advantage of CiviCRM's php autoloader, this file should be named
`path/to/myextension/CRM/Myextension/APIWrapper.php`

```php
    class CRM_Myextension_APIWrapper implements API_Wrapper {
      /**
       * the wrapper contains a method that allows you to alter the parameters of the api request (including the action and the entity)
       */
      public function fromApiInput($apiRequest) {
        if ('Invalid' == CRM_Utils_Array::value('contact_type', $apiRequest['params'])) {
          $apiRequest['params']['contact_type'] = 'Individual';
        }
        return $apiRequest;
      }

      /**
       * alter the result before returning it to the caller.
       */
      public function toApiOutput($apiRequest, $result) {
        if (isset($result['id'], $result['values'][$result['id']]['display_name'])) {
          $result['values'][$result['id']]['display_name_munged'] = 'MUNGE! ' . $result['values'][$result['id']]['display_name'];
          unset($result['values'][$result['id']]['display_name']);
        }
        return $result;
      }
    }
```
