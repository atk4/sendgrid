Welcome to SendGrid + ATK add-on. With `atk4/sendgrid` you can integrate your application or back-end service with email sending service provided by SendGrid.

## Features

-   Implements `Outbox` protocol (https://github.com/atk4/protocols), so any application or module that needs to send out any type of information can use class `\atk4\sendgrid\Outbox` to schedule and send out messages. 
-   Contains `MailTemplate` model which is linked up to https://sendgrid.com/docs/API_Reference/Web_API_v3/Transactional_Templates/templates.html for storing your templates in SendGrid itself.
-   Support for updating deliverability progress in the `Outbox`

## Installation

To install, require this module through composer, then execute:

``` php
$outbox = $app->add(new \atk4\sendgrid\Outbox(<your config>));
$outbox->setUp();
```

Note, that if your configuration change, you may need to execute `setUp` again.

## Usage

You can use `outbox` methods like this:

``` php
$message = $outbox->newMessage( 'john@example.com' );

$message['subject'] = 'test message';
$message['body'] = 'thank you!';

$message->send();
```

The $message extends `\atk4\data\Model` therefore you can save it as a draft into any persistence.

### Templates

In most cases you would want to work with the templates.

``` php
$message = $outbox->newMessage( 'john@example.com', 'password_reminder' );

$message->tag['hash'] = $hash;
$message->send();
```

You can edit templates either in SendGrid or locally in you application through model `atk4\sendgrid\Template`:

``` php
$layout->add('CRUD')->setModel(new \atk4\sendgrid\Template($app->outbox));
```

### Delayed send

In some cases you would like to avoid delay related to sending out messages:

``` php
$message->sendLater();
```

This will place message into `Outbox`, which can then be scheduled to send out automatically.

``` php
$outbox->sendQueue();
```

You can actually access the queue directly:

``` php
$last_mail = $outbox->refMessages()
  ->addCondition('to', 'john@example.com')
  ->setOrder('date desc')
  ->loadAny();

echo "Status of last message is ".$last_mail['status'];
```

### User-specific Messages

The model `\atk4\sendgrid\Message` can be extended to incorporate per-system or per-user dependency.

``` php
class UserMessage extends \atk4\sendgrid\Message {
	function init() {
      	parent::init();
      
      	$this->hasOne('user_id', [
          	new User(),
          	'default'=>$this->app->user->id
        ]);
	}
  
    // override
    function from($user) {
      	$this['from'] = $user['email'];
        $this['from_name'] = $user['name'];
        $this['user_id'] = $user->id;
        return $this;
    }
}
```

You can make this message class a default:

``` php
$outbox->message_class = new UserMessage();
```

Now sending any messages will be user-dependant:

``` php
$message = $outbox->newMessage( 'john@example.com', 'password_reminder' );

$message->from($app->user);
$message->tag['hash'] = $hash;
$message->send();
```

Also you can easily find messages sent by a specific user:

``` php
$user_mail = $outbox->refMessages()
  ->addCondition('user_id', $this->app->user->id);

$this->add('Grid')->setModel($user_mail);
```



