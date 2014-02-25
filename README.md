ImapPlugin
==========
 
Imap datasource for CakePHP. 
Requires CakePHP 2.4

This is a cake-2.4 supporting version of kvz's cake-email-plugin
https://github.com/kvz/cakephp-emails-plugin/tree/cake-2.0
Additional support correct formating of modern Imap service calls and setting up datasource credentials on the fly is added.

Warning
============
This datasource automatically closes the imap connection with `CL_EXPUNGE`.
This means that if you `->delete()` mails, they will really be gone
from your mailserver, and are not just marked for deletion.

Todo
================

 - Better conditions support  
   'Complex' conditions such as: NOT, OR, arrays as values
   Full support for all the imap_search keys
   Send emails and append it to the mailbox
   create new folders
 - 
 

Install
============

 - Unpack / Clone / Copy so that my files are at `app/Plugin/MyEmail/*` (Assumming MyEmail is your plugin name)

Config
========

 - Edit your `database.php` file like so:

```php
<?php
class DATABASE_CONFIG {
    // ... your normal database config here ...
    // Imap email connection
    public $emailTicket = array(
        'datasource' => 'MyEmail.ImapSource',
        'server' => 'imap.gmail.com',
        'connect' => 'novalidate-cert',
        'username' => 'tickets',
        'password' => 'xxxxxxxxxx',
        'port' => '993',
        'tail' => '[Gmail]/All Mail',
        'ssl' => true,
        'encoding' => 'UTF-8',
        'error_handler' => 'php',
        'auto_mark_as' => array(
            'Seen',
            // 'Answered',
            // 'Flagged',
            // 'Deleted',
            // 'Draft',
        ),
    );
}
```
OR

Implement
===========

```php
<?php
class TicketEmail extends AppModel {
    // Important:
    public $useDbConfig = 'emailTicket';
    public $useTable    = false;

    // Whatever:
    public $displayField = 'subject';
    public $limit        = 10;

    // Semi-important:
    // You want to use the datasource schema, and still be able to set
    // $useTable to false. So we override Cake's schema with that exception:
    function schema ($field = false) {
        if (!is_array($this->_schema) || $field === true) {
            $db =& ConnectionManager::getDataSource($this->useDbConfig);
            $db->cacheSources = ($this->cacheSources && $db->cacheSources);
            $this->_schema = $db->describe($this, $field);
        }
        if (is_string($field)) {
            if (isset($this->_schema[$field])) {
                return $this->_schema[$field];
            } else {
                return null;
            }
        }
        return $this->_schema;
    }
}
```

Integrate
===========

Here are a couple of supported examples:

```php
<?php
// Get many
$ticketEmails = $this->TicketEmail->find('all', array(
    // Set recursive to 1 to also get attachments,
    // presented in a separate [Attachment][0] model
    'recursive' => -1,
    'conditions' => array(
        'answered' => 0,
        'deleted' => 0,
        'draft' => 0,
        'flagged' => 0,
        'recent' => 0,
        'seen' => 0,
    ),
));
pr(compact('ticketEmails'));

// Get one
$ticketEmail = $this->TicketEmail->find('first', array(
    'conditions' => array(
        'id' => 21879,
    ),
));
print_r(compact('ticketEmail'));

// Get subject
$this->TicketEmail->id = 21879;
$subject = $this->TicketEmail->field('subject');
print(compact('subject'));


// Delete one
$this->TicketEmail->delete(21879);

// Delete many
$this->TicketEmail->deleteAll(array(
    'deleted' => 0,
    'seen' => 0,
    'from' => 'kevin@true.nl',
));
```
Changes
===========

Now you can set your credentials on the Fly.
Change the database.php as follows

```php
<?php
class DATABASE_CONFIG {
    // ... your normal database config here ...
    // Imap email connection
    public $emailTicket = array(
        'datasource' => 'MyEmail.ImapSource',
        'ssl' => true,
        'encoding' => 'UTF-8',
        'error_handler' => 'php',
        'auto_mark_as' => array(
            'Seen',
            // 'Answered',
            // 'Flagged',
            // 'Deleted',
            // 'Draft',
        ),
    );
}
```

And you can modify the controller as

```php
<?php
$dataSource = ConnectionManager::getDataSource('emailTicket');        
//set the imap config
$dataSource->config['server'] = 'imap.gmail.com';
$dataSource->config['connect'] = 'novalidate-cert';
$dataSource->config['port'] = '993';
$dataSource->config['ssl'] = true;
$dataSource->config['tail'] = '[Gmail]/All Mail';    
$dataSource->config['username'] = 'username@gmail.com';
$dataSource->config['password'] = 'xxxxxxxxxx';
$ticketEmails = $this->TicketEmail->find('all', array(
    // Set recursive to 1 to also get attachments,
    // presented in a separate [Attachment][0] model
    'recursive' => -1,
    'conditions' => array(
        'answered' => 0,
        'deleted' => 0,
        'draft' => 0,
        'flagged' => 0,
        'recent' => 0,
        'seen' => 0,
    ),
));
pr(compact('ticketEmails'));

```