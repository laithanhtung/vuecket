# vuecket
Power of Vue.JS married with magic of Apache Wicket

Vuecket allows to be reactive on frontend and backend without coding of REST services.

1. [Progress and Plans](#current-progress-and-plans)
2. [Enabling Vuecket](#enabling-vuecket)
3. [Association of Wicket and Vue Components](#association-of-wicket-and-vue-components)
4. [Server-side methods](#server-side-methods)

## Current Progress and Plans

- [X] Loading of Vue components code
   - [X] From JSON configuration
   - [X] From Vue files
   - [X] From NPM packages
- [ ] Support of propogation of Vue events to server side
- [ ] Support of data channels between server side and client
   - [ ] One Time - upon Vue component load
   - [ ] Periodical refresh from server side
   - [ ] WebSocket based refresh from server side
- [X] Support of server based Vue methods

## Enabling Vuecket

Add the following dependency into your `pom.xml`:

```xml
<dependency>
   <groupId>org.orienteer.vuecket</groupId>
   <artifactId>vuecket</artifactId>
   <version>${project.version}</version>
</dependency>
```

## Association of Wicket and Vue Components

To start using of Vuecket power you should associate your server-side component(Wicket) and client-side component(Vue.js).
You have 2 ways how to do that: 
* either through Annotations 
* or java code.

Vue.js components can be also defined by:
* JSON description
* VUE file
* NPM package

### Annotations

The following code will allow you to make from common Wicket Label component which supports Markdown
```java
@VueNpm(packageName = "vue-markdown", path = "dist/vue-markdown.js", enablement = "Vue.use(VueMarkdown)")
public class VueMarkdown extends Label {

	public VueMarkdown(String id, IModel<?> model) {
		super(id, model);
	}

	public VueMarkdown(String id, Serializable label) {
		super(id, label);
	}

	public VueMarkdown(String id) {
		super(id);
	}
	
	@Override
	protected void onComponentTag(ComponentTag tag) {
		tag.setName("vue-markdown");
		super.onComponentTag(tag);
	}
}
```

Check the following annotations: `@VueJson`, `@VueFile` and `@VueNpm`

### Directly on Wicket Componet
```java
add(new VueComponent<String>("app")
      .setVueDescriptor("{ data: { message : 'Hello Vue'}}")
);

add(new VueComponent<String>("app2")
      .setVueDescriptor(new PackageResourceReference(HomePage.class,"HomePage.app2.vue"))
);

add(new Label("app3").add(new VueBehavior(new VueJsonDescriptor("{ data: { message : 'Hello Vue'}}"))));
```

## Server-side methods

Vuecket can work transparantly for Vue code. But you can add more spice by invoking server based methods from your Vue code.

There are 2 ways how you can use Vuecket server methods:

* `vcInvoke` - asynchronous invokation of server method. No reply from server expected. But server side method has possiblity to "push" some changes to the client side, if needed.
* `vcCall` - return Promise which will contain response from server side 

Example from test Vuecket application:

```java
add(new VueComponent<Object>("app5") {
	@VueMethod("count")
	public void updateCountModel(IVuecketMethod.Context ctx, int count) {
		IVuecketMethod.pushDataPatch(ctx, "server", "Hello from server #"+count);
	}
}.setVueDescriptor("{ data: { count : 0, server: 'Hello from client side' }}"));
```

```html
<div>
	<h1>App #5</h1>
	<div wicket:id="app5">
		<button @click="count++; vcInvoke('count', count)">Clicked {{count}} times</button>
		{{ server }}
	</div>
</div>
```
