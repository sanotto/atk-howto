# Working with views
ATK uses smarty to render it's templates. Right out of the box, ATK does not have any mechanism to let you render customized views.
The templates used to render the application are usually stored in **vendor/sintattica/atk/src/Resources/templates**, a folder that 
should be read only as it is updated by composer.
So, what if you need to render something out of the ordianry CRUD user interface that ATK provides? It can be done.

## Adding a "renderView" method.

Let's supose you have a notifications node, like this:

```php
class notifications extends Node
{
        function __construct($nodeUri, $flags=null)
        {
                $this->table_name="notifications";
                parent::__construct($nodeUri, $flags | null|Node::NF_ADD_LINK);

                $this->setTable($this->table_name);
                $this->addFlag(Node::NF_ADD_LINK);
                $this->add(new Attribute('id', A::AF_AUTOKEY));
                $this->add(new Attribute('tittle', A::AF_SEARCHABLE|A::AF_OBLIGATORY | A::AF_OBLIGATORY|A::AF_SEARCHABLE, 50), NULL, 10);                
                $this->add(new textattribute('notification', A::AF_OBLIGATORY|A::AF_HIDE_LIST), NULL, 60);
        }
 }       
```

And you want to render this like an unordered list, like this:

* Notification 1
* Notification 2

We will add a method to our node, the **renderView** method:

```php
use Sintattica\Atk\Core\Atk;

class notifications extends Node
{
    function __construct($nodeUri, $flags=null)
    {
                $this->table_name="notifications";
                parent::__construct($nodeUri, $flags | null|Node::NF_ADD_LINK);

                $this->setTable($this->table_name);
                $this->addFlag(Node::NF_ADD_LINK);
                $this->add(new Attribute('id', A::AF_AUTOKEY));
                $this->add(new Attribute('tittle', A::AF_SEARCHABLE|A::AF_OBLIGATORY | A::AF_OBLIGATORY|A::AF_SEARCHABLE, 50), NULL, 10);                
                $this->add(new textattribute('notification', A::AF_OBLIGATORY|A::AF_HIDE_LIST), NULL, 60);
    }

    public function renderView($template, $data, $module='')
    {
         $dir = Atk::getInstance()->moduleDir($this->getModule());
         $node_name = $this->getType();
         $fullpath = $dir.
                        DIRECTORY_SEPARATOR.
                         'views'.
                         DIRECTORY_SEPARATOR.
                         $node_name.
                         DIRECTORY_SEPARATOR.
                         $template;

         $ui = $this->getUi();
         #$output= Tools::href(Tools::dispatch_url("Security.Users", "admin"),"<span class='glyphicon glyphicon-print'></span> ")
         $box =  $ui->render($fullpath, $data, $module);
         $page = $this->getPage();
         $page->addContent($box);
    }
}
```

Please, note the **use Sintattica\Atk\Core\Atk;** at the beginning of the file, it is necessary for the **Atk::getInstance()** to resolve 
properly.

## Defining the "view"

In your module's folder, let's say **src/modules/notifications** add a "views" folder i.e. **src/modules/notifications/views**.
In that folder add another folder for each node in your module, i.e. **src/modules/notifications/views/notification**.
Now we can store the custom views for our notification node as this **src/modules/notifications/views/notification/render_list.tpl**.
The template is in smarty style, so our render_list.tpl could look like:

```smarty
<ul>
{foreach from=$notifications item=notification}
   <li> class="panel-title"> {$notification.title}</li>
{/foreach}
</ul>
```

So we can now write an action called  show_list, like this:

```php
      public function action_display()
      {
          $notifications = $this->select($filter)
                                  ->getAllRows();
          $to_show = [];
          foreach($notifications as $notification)
          {
              //Here filter if notification should or should not be displayed
              if ($display)
              {
                $to_show['notifications'][]=$notification;
              }
          }
 
          $this->renderView('render_list.tpl', $to_show);
      }

```
