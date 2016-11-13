---
layout: post
title:  "Manually rendering status messages in Drupal 8"
date:   2016-11-13 15:05:22 +0100
categories: drupal8
---
Often when it comes to extreme customisation in Drupal8 layouts, you need to have the freedom
to render Drupal status messages in a manual way, without using the pre-defined Twig variables
that the CMS fills in the template files.

One example can be a custom controller that passes the pre-rendered status messages to theme
function, that will lately print the messages in your custom Twig template.

When you meet this need, don't dispair! You can achieve this quite easily using the Drupal8
renderer service.

The renderer service is somethig that you rarely use directly, since it's more often called
internally by Drupal under the hood to render, for example, all the renderable arrays that
you are used to see almost everywhere in Drupal code.

Let's suppose you have a custom module named `my_messages` that exposes a controller.
We will pass the renderd status messages to a custom theme function that will take care of
printing these messages exactly wherever you want in your custom template.

First, let's define the theme function. In `my_messages.module` you will have something
like this:

{% highlight php %}
<?php

function my_messages_theme($existing, $type, $theme, $path) {
  return [
    'my_messages_custom' => [
      'template' => 'my_messages_custom',
      'variables' => [
        'messages' => null,
      ],
    ],
  ];
}
{% endhighlight %}

Then we need to have a controller that returns a response. I will skip all the foundations
needed in order to have a working controller (routing file etc.), because it's not the aim
of this post. Your controller will look like this:

{% highlight php %}
<?php

namespace Drupal\my_messages\Controller;

use use Drupal\Core\Controller\ControllerBase;
use Drupal\Core\Render\Renderer;

class MyMessagesController extends ControllerBase {

  public function __construct(Renderer $renderer) {
    $this->renderer = $renderer;
  }

  public static function create(ContainerInterface $container) {
    $renderer = $container->get('renderer');

    return new static($renderer);
  }

  // This function name is up to you, it's
  // driven by the module routing file.
  public function editUserProfile() {
    // This is the magic step. Drupal reads '#type' => 'status_messages'
    // when it's time to render this renderable array and triggers all
    // the rendering chain that first fetches the messages from the
    // session and then it uses its internal theme functions to render
    // them properly.
    $messages = [
      '#type' => 'status_messages',
    ];

    return [
      '#theme' => 'my_messages_custom',
      '#messages' => $this->renderer->render($messages),
    ];
  }

}
{% endhighlight %}

Lastly, in your twig file (somewhere in your theme `my_messages_custom.html.twig`) you must print
these messages, by putting `{% raw %}{{ messages }}{% endraw %}` in the right place for you.
