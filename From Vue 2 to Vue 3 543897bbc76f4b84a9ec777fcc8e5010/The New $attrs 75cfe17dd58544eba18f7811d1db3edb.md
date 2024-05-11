# The New $attrs

Propriétaire: Hugo

Welcome back!

In this lesson we’re going to explore the changes that Vue 3 brings to the `$attrs` of a component.

Understanding these changes is fundamental to the creation of any type of custom components in Vue 3, as the new `$attrs` not only contains your HTML attributes like it did before, but now also your listeners, classes, and styles.

By learning these new concepts, you will be able to understand how attributes are passed down to ANY component in Vue 3, as well as how to correctly bind events from a parent to a child component.

---

## **Differences between Vue 2 and Vue 3 in `$attrs`**

In Vue 2, `$attrs` is a property of a component’s data that includes all of the attributes that the parent passes into a component, with the exception of `class`, and `style`. These two in particular reside at the same level as `$attrs` in the `data` object of a component instance.

Also in Vue 2, we had a `$listeners` object, which included all of the functions that would act as listeners when the child emitted an event.

So what exactly can we find inside `$attrs` now in Vue 3?

In Vue 3, we no longer have the `$listeners` part of the data object. This object has been fully merged into the `$attrs` object, with a few key differences.

Listeners are no longer listed as the exact keyword, such as `click`, or `input` like they were in Vue 2 under `$listeners`.

```html
// Vue 2
<MyButton
  @click="handleClick"
  @custom="handleCustom"
  v-model="value"
  type="button"
  class="btn"
/>

```

```jsx
// Inside MyButton.vue
$listeners = {
  click: handleClick,
  custom: handleCustom,
  input: () => {}
}

$attrs = {
  type: 'button'
}

```

Instead we can find our listeners with their `onEvent` format, just like in vanilla JavaScript.

```html
// Vue 3
<MyButton
  @click="handleClick"
  @custom="handleCustom"
  v-model="value"
  type="button"
  class="btn"
/>

```

```jsx
// Inside MyButton.vue
$attrs = {
  class: 'btn'
  type: 'button',
  onClick: handleClick,
  onCustom: handleCustom,
  'onUpdate:modelValue': () => { value = payload },
}

```

As you can see, our `click` listener can now be found inside $attrs as `onClick`. A `custom` event would be `onCustom`, and even our `v-model` now shows `onUpdate:modelValue` with the default bindings that we learned on lesson 2 in this course.

Inside `$attrs`, you may also find all the attributes that a component receives from the parent, such as `id`, `aria` attributes, `data` attributes and other HTML attributes like `col`, `row`, `type` and `src`. In the case of this example button, it will receive the `type` attribute that our parent injected into it.

It is important to remember that in Vue 3, `class` and `style` are also part of the `$attrs` object.

---

## **Binding $attrs to a component**

In order to better learn how to correctly use Vue 3 `$attrs` in a component, we’re going to build upon a plain input wrapper component. I’ve gone ahead and created the starting stub code, so that we can dive right into it.

The component includes both a `label` and an `input` element — they are both wrapped up inside a parent `div` as we usually would do in a Vue 2 application. Don’t worry, we’ll cover how to remove this unnecessary `div` with multiroot component in the next lesson.

The `label` is bound to a `label` property, and the `input` element is `v-model` capable with the defaults: `modelValue` as a property, and `update:modelValue` as the emitted event.

📃**BaseInput.vue**

```html
<template>
  <div>
    <label>{{ label }}</label>
    <input
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
    />
  </div>
</template>

<script>
export default {
  props: {
    modelValue: {
      type: [String, Number],
      default: ''
    },
    label: {
      type: String,
      default: null
    }
  }
}
</script>

```

We can now use the `BaseInput` component directly in `App.vue` by importing it and adding it to the `template`.

📃**App.vue**

```html
<template>
  <div id="app">
    <BaseInput
      v-model="email"
      label="Email:"
    />

    <pre>{{ email }}</pre>
  </div>
</template>

<script>
import { ref } from 'vue'
import BaseInput from './components/BaseInput'

export default {
  name: 'App',
  components: {
    BaseInput
  },
  setup () {
    const email = ref('')

    return {
      email
    }
  }
}
</script>

```

Now that we have a component to play with, let’s take a look at what happens when we try to apply a `type` declaration to our component instance in `App.vue` in order to change this `input` to a type of `email`.

📃**App.vue**

```html
<BaseInput
  v-model="email"
  label="Email:"
  type="email"
/>

```

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.jpg?alt=media&token=fe49fd01-a5c7-4d17-813f-3ae0d4af35b7](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.jpg?alt=media&token=fe49fd01-a5c7-4d17-813f-3ae0d4af35b7)

As expected, we get the same behaviour as in Vue 2. The `type` declaration has been set into the root `div` tag of the component.

If we now add a `class` declaration to our `BaseInput` instance inside `App.vue`, we will also get the same behaviour.

📃**App.vue**

```html
<BaseInput
  v-model="email"
  label="Email:"
  type="email"
  class="thicc"
/>

```

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.jpg?alt=media&token=a2ebbebd-27e1-45fa-a9e3-486179c0fbae](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.jpg?alt=media&token=a2ebbebd-27e1-45fa-a9e3-486179c0fbae)

Let’s go back to `BaseInput.vue` and bind the `$attrs` correctly into our `input` element.

📃**BaseInput.vue**

```html
<template>
  <div>
    <label>{{ label }}</label>
    <input
      v-bind="$attrs"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
    />

    <pre>{{ $attrs }}</pre>
  </div>
</template>

```

Now if we check the browser, we’ll notice an interesting thing.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.jpg?alt=media&token=48d14fe9-8a17-4232-8b31-835c6f3c28cf](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.jpg?alt=media&token=48d14fe9-8a17-4232-8b31-835c6f3c28cf)

The `input` element is now correctly binding our `type`, but *also* the `class`! Although this may not seem very exciting at face value, this is actually a huge improvement in Vue 3 over Vue 2.

In Vue 2, not only were the `class` and `style` tags not declared inside `$attrs` — they were always fixed into the root of the component, in this case, our `div`. In Vue 3 we can use this ability to bind classes into components without having to resort to less-than-ideal solutions like creating a `classes` property.

Did you notice? Our `class` and `type` are still bound to the wrapping `div` as well —`inheritAttrs: false` is still a thing.

We need to add this to our component’s object in order to let Vue 3 know that we are going to take care of binding everything ourselves, so let’s go ahead and add it to our component.

📃**BaseInput.vue**

```jsx
export default {
  inheritAttrs: false,
  props: {
    modelValue: {
      type: [String, Number],
      default: ''
    },
    label: {
      type: String,
      default: null
    }
  }
}

```

If we check the browser one more time, now the `div` will not contain the bindings that we’re setting for our `input`.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.opt.jpg?alt=media&token=02d6f4d2-8831-49b1-a4e9-99cacccdfb9f](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.opt.jpg?alt=media&token=02d6f4d2-8831-49b1-a4e9-99cacccdfb9f)

---

## **Binding listeners through our $attrs**

Now that we have our `$attr` binding set up, we can take a look at how to bind `listeners` into our component.

We currently have our component set up to listen directly to `input` events through the `@input` binding. But sometimes we want to allow our component to respond to any type of event that the parent wants to listen to, especially in the case of binding into native input elements.

For example, let’s go back to `App.vue` and add an event listener for `blur` into our `BaseInput` instance that will change the value of `email` whenever the user *blurs* or tabs out of the input field.

📃**App.vue**

```html
<BaseInput
  v-model="email"
  @blur="email = 'blurrr@its.cold'"
  label="Email:"
  type="email"
  class="thicc"
/>

```

Let’s go back to the browser and test it out by blurring out of the input element.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.opt.jpg?alt=media&token=6324b915-db5c-4494-a556-96b37a61ef72](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F5.opt.jpg?alt=media&token=6324b915-db5c-4494-a556-96b37a61ef72)

If you had a lot of Vue 2 experience at this point you may be wondering, wait—how?

In Vue 2 we had to add `v-on="$listeners"` into our `input` element to tell Vue that we wanted to delegate all the events (other than the ones that have been manually declared and therefore overridden) to our parent. This would allow our parent, `App.vue` in this case, to handle declaring the logic for when these events happened—like we just did with the `blur` listener.

In Vue 3, this is no longer necessary. The moment we added our `v-bind="$attrs"` declaration we also added a binding for all the listeners that the parent passed into our component.

Remember, `$attrs` now also includes all the listeners, so in this example it will contain an `onBlur` declaration that holds the function that sets `email` to `blurrr@its.cold`

How convenient is that? 😎

---

## **Last minute cleanup**

If you’re like me and you like to keep your `v-bind` clean and contained, let me tell you that there is a nice way to refactor the `@input` listener into our `v-bind` declaration using the JavaScript spread operator.

We currently have our `template` inside `BaseInput` set like this.

📃**BaseInput.vue**

```html
<input
  v-bind="$attrs"
  @input="$emit('update:modelValue', $event.target.value)"
  :value="modelValue"
/>

```

Since `$attrs` is an object, we can safely use the spread operator to wrap it inside an object. This way, we can now declare the `input` listener as `onInput` inside our `v-bind` declaration to keep all of our internal listener’s logic inside of it.

📃**BaseInput.vue**

```html
<input
  v-bind="{
    ...$attrs,
    onInput: (event) => $emit('update:modelValue', event.target.value)
  }"
  :value="modelValue"
/>

```

Not only is this clearer to look at, but it’s also a very good tool to make sure that certain events like `input` don’t get overwritten by the parent.

---

## **Wrapping up**

In this lesson we learned how to correctly bind our `$attrs` object in Vue 3, and the differences that it has with Vue 2. We also learned how to bind our `listeners` now that the `$listeners` object has been merged into `$attrs`.

In our next lesson, we will take a deeper look into single vs multiple root components and what role `$attrs` plays when migrating from single root to multi root.

See you there!