Using Form Wizard
=================

Form wizards, or multi-step forms, are a process by which the user goes through 
or can use an arbitrary number of different forms to create a single object or 
perform a single task.

The form wizard tool allows a single entry point to easily set up a wizard of 
multiple forms, provide callbacks to handle data storage and updates between 
forms and when forms are completed.


The form info array
-------------------

The wizard starts with an array of data that describes all of the forms 
available to the wizard and sets options for how the wizard will present and 
control the flow. Here is an example of the $form_info array as used in the 
my_form module: 

```
  $form_info = array(
    'id' => 'my_form_page',
    'path' => "admin/structure/pages/edit/$page_name/%step",
    'show trail' => TRUE,
    'show back' => TRUE,
    'show return' => FALSE,
    'next callback' => 'my_form_page_add_subtask_next',
    'finish callback' => 'my_form_page_add_subtask_finish',
    'return callback' => 'my_form_page_add_subtask_finish',
    'cancel callback' => 'my_form_page_add_subtask_cancel',
    'order' => array(
      'basic' => t('Basic settings'),
      'argument' => t('Argument settings'),
      'access' => t('Access control'),
      'menu' => t('Menu settings'),
      'multiple' => t('Task handlers'),
    ),
    'forms' => array(
      'basic' => array(
        'form id' => 'my_form_page_form_basic'
      ),
      'access' => array(
        'form id' => 'my_form_page_form_access'
      ),
      'menu' => array(
        'form id' => 'my_form_page_form_menu'
      ),
      'argument' => array(
        'form id' => 'my_form_page_form_argument'
      ),
      'multiple' => array(
        'form id' => 'my_form_page_argument_form_multiple'
      ),
    ),
  );
  ```
  
The above array starts with an id which is used to identify the wizard in 
various places and a path which is needed to redirect to the next step between 
forms. It then creates some settings which control which pieces are displayed. 
In this case, it displays a form trail and a 'back' button, but not the 'return' 
button. Then there are the wizard callbacks which allow the wizard to act 
appropriately when forms are submitted. Finally it contains a list of forms and 
their order so that it knows which forms to use and what order to use them by 
default. Note that the keys in the order and list of forms match; that key is 
called the step and is used to identify each individual step of the wizard. 

The Form Array
-------------------


Here is a full list of every item that can be in the form info array:

id
An id for wizard. This is used like a hook to automatically name callbacks, as 
well as a form step's form building function. It is also used in trail theming. 

path
The path to use when redirecting between forms. %step will be replaced with the 
key for the form.
return path
When a form is complete, this is the path to go to. This is required if the 
'return' button is shown and not using AJAX. It is also used for the 'Finish' 
button. If it is not present and needed, the cancel path will also be checked.
cancel path
When a form is canceled, this is the path to go to. This is required if the 
'cancel' is shown and not using AJAX.
show trail
If set to TRUE, the form trail will be shown like a breadcrumb at the top of 
each form. Defaults to FALSE.
show back
If set to TRUE, show a back button on each form. Defaults to FALSE.
show return
If set to TRUE, show a return button. Defaults to FALSE.
show cancel
If set to TRUE, show a cancel button. Defaults to FALSE.
back text
Set the text of the 'back' button. Defaults to t('Back').
next text
Set the text of the 'next' button. Defaults to t('Continue').
return text
Set the text of the 'return' button. Defaults to t('Update and return').
finish text
Set the text of the 'finish' button. Defaults to t('Finish').
cancel text
Set the text of the 'cancel' button. Defaults to t('Cancel').
ajax
Turn on AJAX capabilities. Defaults to FALSE.
modal
Put the wizard in the modal tool. The modal must already be open and called 
from an ajax button for this to work, which is easily accomplished using 
functions provided by the modal tool.
ajax render
A callback to display the rendered form via ajax. This is not required if 
using the modal tool, but is required otherwise since ajax by itself does 
not know how to render the results. Params: &$form_state, $output.
finish callback
The function to call when a form is complete and the finish button has been 
clicked. This function should finalize all data. Params: &$form_state. Defaults 
to $form_info['id']._finish if function exists.
cancel callback
The function to call when a form is canceled by the user. This function should 
clean up any data that is cached. Params: &$form_state. Defaults to 
$form_info['id']._cancel if function exists.
return callback
The function to call when a form is complete and the return button has been 
clicked. This is often the same as the finish callback. Params: &$form_state. 
Defaults to $form_info['id']._return if function exists.
next callback
The function to call when the next button has been clicked. This function 
should take the submitted data and cache it for later use by the finish 
callback. Params: &$form_state. Defaults to $form_info['id']._next if function 
exists.
order
An optional array of forms, keyed by the step, which represents the default 
order the forms will be displayed in. If not set, the forms array will control 
the order. Note that submit callbacks can override the order so that branching 
logic can be used.
forms
An array of form info arrays, keyed by step, describing every form available 
to the wizard. If order array isn't set, the wizard will use this to set the 
default order. Each array contains:
form id
The id of the form, as used in the Backdrop form system. This is also the name 
of the function that represents the form builder. Defaults to 
$form_info['id']._.$step._form.
include
The name of a file to include which contains the code for this form. This makes 
it easy to include the form wizard in another file or set of files. This must 
be the full path of the file, so be sure to use backdrop_get_path() when setting 
this. This can also be an array of files if multiple files need to be included.
title
The title of the form, to be optionally set via backdrop_get_title. This is 
required when using the modal if $form_state['title'] is not set.

Invoking the form wizard
-------------------

Your module should create a page callback via hook_menu, and this callback 
should contain an argument for the step. The path that leads to this page 
callback should be the same as the 'path' set in the $form_info array.

The page callback should set up the $form_info, and figure out what the default 
step should be if no step is provided (note that the wizard does not do this 
for you; you MUST specify a step). Then invoke the form wizard:

  ```
  $form_state = array();
  $output = form_wizard_multistep_form($form_info, $step, $form_state);
  ```
  
If using AJAX or the modal, This part is actually done! If not, you have one 
more small step:
  ```
  return $output;
  ```
  
Forms and their callbacks
-------------------

Each form within the wizard is a complete, independent form using Backdrop's Form 
API system. It has a form builder callback as well as submit and validate 
callbacks and can be form altered. The primary difference between these forms 
and a normal Backdrop form is that the submit handler should not save any data. 
Instead, it should make any changes to a cached object (usually placed on the 
$form_state) and only the _finish or _return handler should actually save any 
real data. 


How you handle this is completely up to you. The recommended best practice is 
to use Backdrop's tempstore, and a good way to do this is to write a couple 
of wrapper functions around the cache that look like these example functions:

  ```
  /**
   * Get the cached changes to a given task handler.
   */
  function my_form_page_get_page_cache($name) {
    $cache = tempstore_get('my_form_page', $name);
    if (!$cache) {
      $cache = my_form_page_load($name);
      $cache->locked = tempstore_test('my_form_page', $name);
    }

    return $cache;
  }

  /**
   * Store changes to a task handler in the object cache.
   */
  function my_form_page_set_page_cache($name, $page) {
    $cache = tempstore_set('my_form_page', $name, $page, REQUEST_TIME + 3600);
  }

  /**
   * Remove an item from the object cache.
   */
  function my_form_page_clear_page_cache($name) {
    tempstore_clear('my_form_page', $name);
  }
  ```

Using these wrappers, when performing a get_cache operation, it defaults to 
loading the real object. It then checks to see if another user has this object 
cached using the tempstore_test() function, which automatically sets a 
lock (which can be used to prevent writes later on). 


With this set up, the _next, _finish and _cancel callbacks are quite simple:

  ```
  /**
   * Callback generated when the add page process is finished.
   */
  function my_form_page_add_subtask_finish(&$form_state) {
    $page = &$form_state['page'];

    // Create a real object from the cache
    my_form_page_save($page);

    // Clear the cache
    my_form_page_clear_page_cache($form_state['cache name']);
  }

  /**
   * Callback generated when the 'next' button is clicked.
   *
   * All we do here is store the cache.
   */
  function my_form_page_add_subtask_next(&$form_state) {
    // Update the cache with changes.
    my_form_page_set_page_cache($form_state['cache name'], $form_state['page']);
  }

  /**
   * Callback generated when the 'cancel' button is clicked.
   *
   * All we do here is clear the cache.
   */
  function my_form_page_add_subtask_cancel(&$form_state) {
    // Update the cache with changes.
    my_form_page_clear_page_cache($form_state['cache name']);
  }
  All that's needed to tie this together is to understand how the changes made it 
  into the cache in the first place. This happened in the various form _submit 
  handlers, which made changes to $form_state['page'] based upon the values set in 
  the form: 


  /**
   * Store the values from the basic settings form.
   */
  function my_form_page_form_basic_submit($form, &$form_state) {
    if (!isset($form_state['page']->pid) && empty($form_state['page']->import)) {
      $form_state['page']->name = $form_state['values']['name'];
    }

    $form_state['page']->admin_title = $form_state['values']['admin_title'];
    $form_state['page']->path = $form_state['values']['path'];

    return $form;
  }
  ```

  No database operations were made during this _submit, and that's a very 
important distinction about this system.

Proper handling of back button
-------------------

When using 'show back' => TRUE the cached data should be assigned to the 
#default_value form property. Otherwise when the user goes back to the previous 
step the forms default values instead of his (cached) input is used.

  ```
  /**
   * Form builder function for wizard.
   */
  function wizardid_step2_form($form, &$form_state) {
    $form_state['my data'] = my_module_get_cache($form_state['cache name']);
    $form['example'] = array(
      '#type' => 'radios',
      '#title' => t('Title'),
      '#default_value' => $form_state['my data']->example ? $form_state['my data']->example : default,
      '#options' => array(
        'default' => t('Default'),
        'setting1' => t('Setting1'),
      ),
    );

    return $form;
  }

  /**
   * Submit handler to prepare needed values for storing in cache.
   */
  function wizardid_step2_form_submit($form, &$form_state) {
    $form_state['my data']->example = $form_state['values']['example'];
  }

  ```

The data is stored in the my data object on submitting. If the user goes back 
to this step the cached my data is used as the default form value. The function 
my_module_get_cache() is like the cache functions explained above.

Required fields, cancel and back buttons
-------------------

If you have required fields in your forms, the back and cancel buttons will not 
work as expected since validation of the form will fail. You can add the 
following code to the top of your form validation to avoid this problem:

  ```
  /**
   * Validation handler for step2 form
   */
  function wizardid_step2_form_validate(&$form, &$form_state) {
    // if the clicked button is anything but the normal flow
    if ($form_state['clicked_button']['#next'] != $form_state['next']) {
      backdrop_get_messages('error');
      form_set_error(NULL, '', TRUE);
      return;
    }
    // you form validation goes here
    // ...
  }
  ```
  
  
Wizard for anonymous users
-------------------

If you are creating a wizard which is be used by anonymous users, you might 
run into some issues with backdrop's caching for anonymous users. You can 
circumvent this by using hook_init and telling backdrop to not cache your wizard 
pages:

  ```

/**
 * Implementation of hook init
 */
function mymodule_init() {
  // if the path leads to the wizard
  if (backdrop_match_path($_GET['q'], 'path/to/your/wizard/*')) {
    // set cache to false
    $GLOBALS['conf']['cache'] = FALSE;   
  }
}
  ```
