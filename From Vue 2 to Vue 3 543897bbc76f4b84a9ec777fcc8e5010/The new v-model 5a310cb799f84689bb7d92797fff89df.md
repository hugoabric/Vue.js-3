# The new v-model

Propriétaire: Hugo

As you probably know,`v-model` allows us to very quickly and easily capture an input’s value into the state of our application. Every time the user types or interacts with an input, `v-model` will let the parent know so that it can update our state.

In Vue 3, `v-model` has gone through a redesign that gives us more power and flexibility when defining how this double binding should be done.

---

## **Kicking it off with Native inputs**

Let’s start by looking at a native input element.

```html
<template>
	<input type="text" />
</template>

```

In Vue 2, whenever you add a `v-model` declaration to a native input element, the compiler produces a block of code that handles the correct input value and event to be listened to.

```html
<input type="text" v-model="myValue" />

export default {
  data() {
    myValue: null
  }
}

```

This strategy works fairly well, but what if our component uses a dynamic value to set the type of input?

I’m sure you’ve been in a situation where you have created an input component that can either be a type of `input` for someone’s name, for example, and that with the change of a property you use it as a type `email` for their email address.

---

## **How does it compile?**

Because Vue 2 cannot “guess” what type of element this is going to be at runtime, due to the possibility of the data changing at any given time, the Vue 2 compiler is forced to output a very lengthy and verbose block of code to handle *every possible scenario*.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2FL2-1.opt.jpg?alt=media&token=335c17d1-1c86-4327-ae2c-a619dd164b82](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2FL2-1.opt.jpg?alt=media&token=335c17d1-1c86-4327-ae2c-a619dd164b82)

Don’t worry, we don’t need to go over every line of code. Just know that it basically has to prepare for every type of possible scenario.

In Vue 3, outputting this amount of code is no longer necessary because `v-model` for input elements behaves almost the same way it does for custom components — with an extra module that helps Vue decide which prop/event to apply in each case.

The compiled result in comparison is incredibly smaller.

```jsx
h('input', {
  modelValue: myValue,
  'onUpdate:modelValue': value => {
    myValue = value
  }
})

```

---

## **The new defaults**

In Vue 3, when creating a component that has `v-model` capabilities we need to use a new set of defaults for creating the `v-model` binding.

In Vue 2, no matter what type of native input you were binding to inside the component you would always bind the value of your data to a `value` property, and you would listen to an `input` event.

Of course there was a way to modify this default behaviour by declaring a `model` property in our Vue component, but that’s the Vue 2 API and we’re not going to look in depth into it.

In Vue 3, we now have two new defaults. For the `prop` that binds the input of the value, we use `modelValue`, and for the emitted event we use `update:modelValue`.

I want you to play close attention to the names of the events before you panic about the verboseness of these new defaults. Did you notice how `modelValue` is actually present in both of them?

The new emit default `update:modelValue` can be extracted into two different parts. The declaration that something is being updated `update:` and the model that is being updated `modelValue`.

This is going to play a very important role later on in the course, when we look at how to create multiple `v-model` bindings into a single component!

Now, let’s take a look at how the code of a simple `input` wrapper component might look with the new Vue 3 `v-model` syntax.

```html
<template>
  <input
    :value="modelValue"
    @input="$emit(
      'update:modelValue',
      $event.target.value
    )"
  >
</template>

<script>
export default {
  props: {
    modelValue: {
      type: [String, Number],
      default: ''
    }
  }
}
</script>

```

---

## **Using the new v-model in component instances**

Now that we have the base for a `v-model` capable component, let’s take a look at how we would use it in an application.

As with any other component, we need to import it and declare it in the `components` object for our parent.

```html
<template>
  <div>

  </div>
</template>

<script>
import BaseInput from './BaseInput'

export default {
  components: { BaseInput }
}
</script>

```

Next, we add the component to our `template` so that we can declare the `v-model` on top of it. After, we are going to use the object syntax (no composition API this time!) and use a `data` object to create a simple reactive state.

```html
<template>
  <div>
    <BaseInput
      v-model="myInput"
    />
  </div>
</template>

<script>
import BaseInput from './BaseInput'

export default {
  components: { BaseInput },
  data() {
    return {
      myInput: ''
    }
  }
}
</script>

```

At this point you’re wondering, we’ll that’s all right and nice, but I already know this!

Fair enough, but I wanted to show you one last thing before we wrapped up our lesson. Using `v-model` like this is now actually a shorthand! The binding has been modified to now accept an intermediate parameter before the binding.

So `v-model="myInput"` is actually now a shorthand for `v-model:modelValue="myInput"`

You’re probably wondering at this point why this is important or actually useful. In the next lesson when we dive into more advanced parts of the new `v-model` system, we’re going to talk about *multi* `v-model` bindings, and this new syntax is going to play a very important role.

---

## **Coming up next**

Now that we understand the basics of the new `v-model` system and its improved bindings, let’s go into the next lesson and look at a couple other new tools that it provides us for component development.

We will finally look at the multi `v-model` capabilities that I’ve been hinting at, and a cool new feature to build our own modifiers.

Can you think of any components in your current Vue 2 applications that will benefit or be able to be enhanced already by these new features?

See you in the next lesson!