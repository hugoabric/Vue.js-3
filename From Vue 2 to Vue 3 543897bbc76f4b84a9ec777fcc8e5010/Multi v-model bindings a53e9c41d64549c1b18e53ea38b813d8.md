# Multi v-model bindings

Propriétaire: Hugo

In our last lesson, we went over the most common way to use `v-model` in a Vue 3 application, and we looked at the changes that the default bindings for input and event listeners bring. These are things we need to know in order to use `v-model` effectively.

It is now time to go deep into one of the advanced changes that Vue 3 brings in terms of `v-model` functionality — multi `v-model` bindings into a single component.

---

## **v-model: Not one, but many**

A really interesting feature of Vue 3’s `v-model` is the ability to add multiple `v-model` bindings to a single component instance.

Have you ever created a complex component that `$emit`s an object as the payload, and inside the object you have a couple of different properties? Maybe you had to pass the parent a lot of information, which forced you to have those properties? Wouldn’t it have been easier if you could have just bound to each of them separately?

Although this approach is fairly common, it brings a bit of overhead in understanding what a component is doing or what you’re supposed to receive out of a `v-model` payload. Instead of being able to quickly see what’s happening in the `template` of a component, you have to dig into the function that receives the payload and read through the implementation to figure out where each of the properties of the object are going to be used.

In Vue 3, with the addition of the multi `v-model` binding, a complex payload is no longer needed.

---

In order to demonstrate multi v-model capabilities, and to better learn how it works, we are going to create a component called `SalutationName`. This component will have two inputs inside of it — a `select` element that will allow the user to select their desired salutation, and an `input` for them to type in their name.

We’ll start out with the template, by adding these two new fields, value bindings, and some attributes.

**📃SalutationName.vue**

```html
<template>
	<div>
	  <select name="salutation">
	    <option value="">-</option>
	     <option
	      v-for="item of salutations"
	      :value="item"
	      :key="item"
	      :selected="salutation === item"
	    >
	      {{ item }}
	    </option>
	  </select>

	  <input
	    :value="name"
	    type="text"
	    name="name"
	  />
	</div>
</template>

```

Now that we have our markdown ready, let’s take a look at our `script`. We need to create a `salutations` array to populate that dropdown, plus declare a couple of `props` that we can bind to the values we already declared.

**📃SalutationName.vue**

```html
<script>
const salutations = [
  'Ms.',
  'Mrs.',
  'Miss',
  'Mx.',
  'Dr.'
]

export default {
  props: {
    salutation: {
      type: String,
      default: ''
    },
    name: {
      type: String,
      default: ''
    }
  },
  setup () {
    return {
      salutations
    }
  }
}
</script>

```

We declare two props, `salutation` and `name`. Both will be of type `String` with an empty string as a default value.

Notice that both these `props` are already binding to their respective elements in the `template`. The `input` through the `value` binding, and the `select` through the `selected` binding in the `option` loop.

In the `setup` function of our component, we are going to return the `salutations` array that we declare at the top of the `script` tag, so that the `select` element can `v-for` loop through it and populate all the `options`.

Finally, let’s add `$emit` calls for each of the inputs to broadcast their respective events.

**📃SalutationName.vue**

```html
<template>
  <div>
    <select
			name="salutation"
			@change="$emit('update:salutation', $event.target.value)"
		>
      <option value="">-</option>
      <option
        v-for="item of salutations"
        :value="item"
        :key="item"
        :selected="salutation === item"
      >
        {{ item }}
      </option>
    </select>

    <input
      :value="name"
      @input="$emit('update:name', $event.target.value)"
      type="text"
      name="name"
    />
  </div>
</template>

<script>
const salutations = [
  'Ms.',
  'Mrs.',
  'Miss',
  'Mx.',
  'Dr.'
]

export default {
  props: {
    salutation: {
      type: String,
      default: ''
    },
    name: {
      type: String,
      default: ''
    }
  },
  setup () {
    return {
      salutations
    }
  }
}
</script>

```

Notice that we are using the `update:X` format for Vue 3’s `v-model` binding syntax that we learned in the last lesson.

For the `select` element, since we’re binding it to the prop `salutation`, we will emit `update:salutation`.

For the `input` element, since we’re binding it to the prop `name`, we will emit `update:name`.

Finally, we can `import` this new component anywhere in our app to use it.

**📃App.vue**

```html
<template>
  <div id="app">
    <SalutationName
      v-model:salutation="form.salutation"
      v-model:name="form.name"
    />

    <pre>{{ form }}</pre>
  </div>
</template>

<script>
import { reactive } from 'vue'
import SalutationName from './components/SalutationName'

export default {
  name: 'App',
  components: {
    SalutationName
  },
  setup () {
    const form = reactive({
      salutation: '',
      name: ''
    })

    return {
      form
    }
  }
}
</script>

```

Take a closer look at the `template` where `SalutationName` is being used. Did you notice the double `v-model` binding?

**📃App.vue**

```html
[...]
<SalutationName
  v-model:salutation="form.salutation"
  v-model:name="form.name"
/>
[...]

```

The first `v-model` declaration is going to provide a two way binding from the property `salutation` of `SalutationName` to our state `form.salutation`.

The second `v-model` declaration is going to provide a two-way binding from the property `name` of `SalutationName` to our state `form.name`.

We can now open this example in our browser. The result is a fully functional double `v-model`ed component!

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2FL3-1.opt.jpg?alt=media&token=c715ff55-056c-4cc1-b9d0-f86a8df1ea90](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2FL3-1.opt.jpg?alt=media&token=c715ff55-056c-4cc1-b9d0-f86a8df1ea90)

---

## **Coming up next**

In this lesson we learned how to make a component ready for multi `v-model` bindings, and how to bind to an instance of this component with multi `v-model`s.

In the next lesson, we’re going to take a look at another cool advanced feature of Vue 3’s `v-model`, the ability to create our own custom model modifiers.

See you in the next lesson!