---
description: A five minute guide to make an editor collaborative
---

# A Collaborative Editor

Yjs is a modular framework for making things collaborative - like editors!

{% hint style="info" %}
If you are impatient jump to the demo at the bottom of the page😉 
{% endhint %}

First, let's decide on an editor to use. There are tons of awesome open-source editors and Yjs supports many of them. 

{% page-ref page="../yjs-ecosystem/editor-bindings/" %}

For the purpose of this guide, we are going to use the [Quill](https://quilljs.com/) editor - a great rich-text editor that is easy to setup. For a complete reference on how to setup Quill I refer to [their documentation](https://quilljs.com/playground/).

{% tabs %}
{% tab title="JavaScript" %}
```javascript
import Quill from 'quill'

const quill = new Quill(document.querySelector('#editor'), {
  modules: {
    cursors: true,
    toolbar: [
      // adding some basic Quill content features
      [{ header: [1, 2, false] }],
      ['bold', 'italic', 'underline'],
      ['image', 'code-block']
    ],
    history: {
      // Local undo shouldn't undo changes
      // from remote users
      userOnly: true
    }
  },
  placeholder: 'Start collaborating...',
  theme: 'snow' // 'bubble' is also great
})
```
{% endtab %}

{% tab title="HTML" %}
```markup
<link href="https://cdn.quilljs.com/1.3.6/quill.snow.css" rel="stylesheet">

<div id="editor" />
```
{% endtab %}

{% tab title="Install" %}
```bash
npm i quill
```
{% endtab %}
{% endtabs %}

Next, we are going to setup Yjs and the [y-quill](../yjs-ecosystem/editor-bindings/yjs-quilljs.md) editor binding.

```bash
npm i yjs y-quill
```

```javascript
import * as Y from 'yjs'
import { QuillBinding } from 'y-quill'

// A Yjs document holds the shared data
const ydoc = new Y.Doc()
// Define a shared text type on the document
const ytext = ydoc.getText('quill')

// "Bind" the quill editor to a Yjs text type.
const binding = new QuillBinding(quill, ytext)
```

The `ytext` type is a shared data type that holds text data and supports formatting attributes \(for example **bold** and _italic_ text\). Yjs automatically resolves concurrent changes so we don't have to worry about conflict resolution anymore. Then we synchronize the `ytext` type with the `quill` editor and keep them in-sync using the [y-quill](../yjs-ecosystem/editor-bindings/yjs-quilljs.md) binding. Almost all editor bindings work like this. So you can simply exchange the editor binding if you choose to use another editor.

But don't stop here, the editor doesn't sync yet to other clients! We need to connect to other peers using a **provider** or [implement our own communication protocol](../tutorials/creating-a-custom-provider.md) to exchange document updates.

{% page-ref page="../yjs-ecosystem/connection-provider/" %}

The[ y-webrtc](../yjs-ecosystem/connection-provider/y-webrtc.md) provider connects clients directly with each other and is a perfect choice for demo applications because it doesn't require you to setup a server.  

{% tabs %}
{% tab title="y-webrtc" %}
```javascript
import { WebrtcProvider } from 'y-webrtc'

const provider = new WebrtcProvider('quill-demo-room', ydoc)

```
{% endtab %}

{% tab title="y-websocket." %}
```javascript
import { WebsocketProvider } from 'y-websocket'

// connect to the public demo server (not in production!)
const provider = new WebsocketProvider('wss://demos.yjs.dev', 'quill-demo-room', ydoc)
```
{% endtab %}

{% tab title="y-dat" %}
```javascript
import { DatProvider } from 'y-dat'

// set null in order to create a fresh
const datKey = '7b0d584fcdaf1de2e8c473393a31f52327793931e03b330f7393025146dc02fb'
const provider = new DatProvider(datKey, ydoc)
```
{% endtab %}

{% tab title="Installation" %}
```bash
npm i y-webrtc # or
npm i y-websocket # or
npm i y-dat
```
{% endtab %}
{% endtabs %}

The providers work similarly to editor bindings. They sync Yjs documents, for example through a  communication protocol or a database. All providers have in common that they use the concept of room-names to connect Yjs documents. In the above example, all documents that specify `'quill-demo-room'` as the room-name will sync.

{% hint style="info" %}
**Providers are meshable.**

You can connect several providers to the Yjs document at the same time. [\(See Tutorial\)](https://jsfiddle.net/dmonad/gh7jm6y5/7/)  
  
You can sync one document through different rooms. For example, to simulate forking behavior.
{% endhint %}

By connecting Yjs with providers and editor bindings we created our first collaborative editor. In the following getting started sections we will explore more Yjs concepts like awareness, shared types, and offline editing.

But for now, let's enjoy what we built. I included the same fiddle twice so you can see the editors sync.



{% embed url="https://jsfiddle.net/dmonad/gh7jm6y5/" %}

{% embed url="https://jsfiddle.net/dmonad/gh7jm6y5/" caption="The complete example in a live demo \(click on \"Result\"\)" %}




