# Sharing code
## Sharing code snippets

When you make a selection in Sourcegraph Editor, you can share it as a code snippet:

<div class="context-menu">
    <img src="/docs/editor/share-code/context-menu.png"></img>
</div>

This will upload the selected code (plus a few surrounding lines) to Sourcegraph as a snippet. You'll receive a private URL that you can share with anyone.

## Sharing discussions

When you create a new discussion, Sourcegraph Editor can also share the selected snippet of code. For example when you share a comment:

<div class="share-comment">
    <img src="/docs/editor/share-code/share-comment.png"></img>
</div>

You'll receive a private URL that you can share with anyone, and they can see the discussion _and_ the code that the discussion is around.

## Configuring your editor

Because both of these features upload the selected code (plus a few surrounding lines) to the Sourcegraph server (specified via the `remote.endpoint` configuration option), you'll need to opt-in to sharing code.

Set `remote.shareContext` to `true` in your user settings:

<div class="configuration">
    <img src="/docs/editor/share-code/configuration.png"></img>
</div>

Note that the code will be sent to the `remote.endpoint` value. In the case of Sourcegraph Enterprise, this will be your enterprise server. By default it is sourcegraph.com.
