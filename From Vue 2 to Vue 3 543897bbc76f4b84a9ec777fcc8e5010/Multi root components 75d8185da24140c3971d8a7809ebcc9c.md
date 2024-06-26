# Multi root components

Propriétaire: Hugo

In our last lesson we learned how `$attrs` bindings work in Vue 3, as well as the removal of `$listeners`, but there’s a little bit more we have to clarify about attribute binding in Vue 3 components before we move on.

Vue 3 allows us the possibility of creating components that have multiple roots, or fragments — note that you may see them also called fragmented root components. This was not possible in Vue 2, so some adjustments to the framework and API were obviously needed.

In this lesson, we will take a look at the differences between single-root and multiple-root components, and Process of transforming a single-root into multi-root.

Understanding these changes, both in how they benefit you as well as potential problems, will allow you to be able to write and debug any component regardless of the template architecture it uses.

---

## **Multi-Root Components in Vue**

We’re going to build upon the last lesson’s component `BaseInput`. In case you’re catching up — here’s the code for the component.

📃**BaseInput.vue**

```html
<template>
  <div>
    <label>{{ label }}</label>
    <input
      v-bind="{
        ...$attrs,
        onInput: (event) => $emit('update:modelValue', event.target.value)
      }"
      :value="modelValue"
    />
  </div>
</template>

<script>
export default {
  inheritAttrs: false,
  props: {
    modelValue: {
      type: [String, Number],
      default: ''
    },
    label: {
      type: String,
      default: ''
    }
  }
}
</script>

```

Inside `App.vue`, we are instantiating it and applying some bindings.

**📃App.vue**

```html
<template>
  <div id="app">
    <BaseInput
      v-model="email"
      @blur="email = 'blurrr@its.cold'"
      label="Email:"
      type="email"
      class="thicc"
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

The result in our browser is a plain email input field and the display of the current `v-model` bindings.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1596471405643.jpg?alt=media&token=0572eda1-b0dd-4861-be4e-f66f95244e2b](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F1.opt.1596471405643.jpg?alt=media&token=0572eda1-b0dd-4861-be4e-f66f95244e2b)

Making this component a multi-root component in Vue 3 is as simple as removing the unnecessary `div` tag that wraps our component. Let’s go ahead and make the change and check in the browser to see if there have been any changes.

📃**BaseInput.vue**

```html
<template>
  <label>{{ label }}</label>
  <input
    v-bind="{
      ...$attrs,
      onInput: (event) => $emit('update:modelValue', event.target.value)
    }"
    :value="modelValue"
  />
</template>

```

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1596471405644.jpg?alt=media&token=bb6e3858-3eab-4cce-b09e-bbae6e0a5533](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F2.opt.1596471405644.jpg?alt=media&token=bb6e3858-3eab-4cce-b09e-bbae6e0a5533)

As you can see, our `BaseInput` is correctly being rendered, the bindings are still in place, and the wrapping `div` tag is nowhere to be found.

I have good news for you! Now that we are working with a multi-root component, we can remove the `inheritAttrs` property completely.

📃**BaseInput.vue**

```html
<template>
  <label>{{ label }}</label>
  <input
    v-bind="{
      ...$attrs,
      onInput: (event) => $emit('update:modelValue', event.target.value)
    }"
    :value="modelValue"
  />
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

Whenever we are working with multi-root components, Vue will no longer attempt to auto-inject attributes into the root node because there isn’t a single one anymore. Since Vue can not safely assume which of these multiple roots should get the attribute fall-through, it will not attempt it at all.

At this point, Vue will check your component file and see if you have `"$attrs"` in your component. If you do, then Vue will assume that you are in control of how the attributes and listeners will be handled; if don’t, you will get a warning.

Let’s go ahead and take out the `v-bind` directive for a moment and take a look at the browser to see what this error looks like.

📃**BaseInput.vue**

```html
<template>
  <label>{{ label }}</label>
  <input
    :value="modelValue"
  />
</template>

```

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.1596471417558.jpg?alt=media&token=292e90b9-f5f7-4f24-81ed-ad2653985eff](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F3.opt.1596471417558.jpg?alt=media&token=292e90b9-f5f7-4f24-81ed-ad2653985eff)

This wording is a little confusing, but what Vue is trying to tell us is:

- Hey, listen. The parent is passing down `type`, `class`, and an event listener (through `v-model`) and I have no idea where to put them

It’s important that we know what this means in case we run into this in one of our projects. Now we know exactly where to look for the problem.

Let’s put back our `v-bind` declaration, though, so that our component behaves correctly.

📃**BaseInput.vue**

```html
<template>
  <label>{{ label }}</label>
  <input
    v-bind="{
      ...$attrs,
      onInput: (event) => $emit('update:modelValue', event.target.value)
    }"
    :value="modelValue"
  />
</template>

```

---

## **The emits property**

If you were looking very closely at the error we generated a bit ago, you may have noticed that one of the warnings stated:

- Extraneous non-emits event listeners (update:modelValue, blur) were passed to component but could not be automatically inherited […]. If the listener is intended to be a component custom event listener only, declare it using the “emits” option.

Vue 3 multi-components introduce a new data level property called `emits`. Let’s check it out.

In more intricate components where you are not declaring a `v-bind="$attrs"` that functions as a catch-all declaration for your listeners, or in `render`-based components where your emits may be generated dynamically, Vue will complain that it cannot find a declaration inside of the component file or template for a custom component that you may be adding.

In these cases, we get a new property called `emits` that sits on the data level right next to others like `components` and `setup`.

This new property is, in its simplest form, an array. So if we were expecting our component to emit an event called `peekedIntoTheBox` we could define it on our component like this.

📃**SchroedingersBox.vue**

```html
<script>
export default {
  emits: ['peekedIntoTheBox']
}
</script>

```

That way, whenever Vue instantiates our component it will know that it may expect an event with the name `peekedIntoTheBox` to be emitted by the component.

A more advanced syntax similar to the one of `props` allows us to even set validators for these declared `emits`.

By using an object format, we can add each of the emitted events as the key of the `emits` object. Then we can define it as `null` to avoid a validator, like this:

📃**SchroedingersBox.vue**

```html
<script>
export default {
  emits: {
    peekedIntoTheBox: null
  }
}
</script>

```

Now to take it a step further, let’s add a validator that checks that the `peekedIntoTheBox` event emits only `dead`, `alive` or `both`. The validator function, just as the ones in `props` should return a boolean value to state whether the payload is valid or not.

If the validation for the emitted value fails, Vue will issue a warning like the following.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.opt.1596471425564.jpg?alt=media&token=ec9c8d69-a039-4bf4-a1ab-ace7e9f1d629](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2F4.opt.1596471425564.jpg?alt=media&token=ec9c8d69-a039-4bf4-a1ab-ace7e9f1d629)

📃**SchroedingersBox.vue**

```html
<script>
export default {
  emits: {
    peekedIntoTheBox: payload => {
      return ['dead', 'alive', 'both'].includes(payload)
    }
  }
}
</script>

```

---

## **Wrapping up**

Multi-root components are a very welcome addition to the Vue toolbox. It will certainly make Vue application `div` nesting nightmares less recurrent.

In this lesson, we learned how to transform single-root components into multi-root, and how to deal with the caveats and quirks of multi-root component attribute fall-through.

We also touched upon an advanced topic, the `emits` property and its correct use.

With this, we wrap up the **From Vue 2 to Vue 3** course. You are now ready to leverage the power of the new Vue 3 capabilities!

Thanks for watching!