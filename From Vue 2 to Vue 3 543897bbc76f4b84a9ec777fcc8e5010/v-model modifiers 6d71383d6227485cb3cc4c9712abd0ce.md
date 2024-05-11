# v-model modifiers

Propriétaire: Hugo

In our last lesson, we learned how to create a component that is capable of multiple `v-model` bindings. We also used this component in our application, and applied two simultaneous bindings into an instance of the component.

This time around, we are going to learn another advanced capability of `v-model` in Vue 3, the ability to create our own custom `v-model` modifiers.

Let’s dive right in.

---

## **v-model: Make it special**

You’ve probably already used `v-model` modifiers before, Vue comes with quite a few out of the box.

- [.lazy](https://vue-docs-next-preview.netlify.app/guide/forms.html#lazy) - listen to `change` events instead of `input` (for native inputs)
- [.number](https://vue-docs-next-preview.netlify.app/guide/forms.html#number) - cast valid input string to numbers
- [.trim](https://vue-docs-next-preview.netlify.app/guide/forms.html#trim) - trim input

Note that all of these are still available for you in Vue 3.

Let’s learn how to create our own modifiers by building upon the example `SalutationName` component that we created in the last lesson.

We are going to first add the ability to pass in a modifier called `capitalize` to both of our bindings.

Here’s the twist:

For the `salutation` binding, we’ll go ahead and capitalize the whole acronym.

For the `name` binding, we’ll just capitalize the first letter.

---

We’ll start by adding the modifiers to our `App.vue`, where the instance of `SalutationName` is being used.

**📃App.vue**

```html
<template>
  <div id="app">
    <SalutationName
      v-model:salutation.capitalize="form.salutation"
      v-model:name.capitalize="form.name"
    />

    <pre>{{ form }}</pre>
  </div>
</template>

[...]

```

Notice that custom modifiers are declared just the same as out-of-the box modifiers, by adding a `.` after the `v-model:model` declaration and the name of the modifier.

Vue 3 is now going to try to inject two new `props` into our component.

1. `salutationModifiers` for the `salutation` v-model binding.
2. `nameModifiers` for the `name` v-model bindings.

Let’s update our `SalutationName.vue` component and add these new props.

**📃SalutationName.vue**

```jsx
[...]
props: {
  salutation: {
    type: String,
    default: ''
  },
  salutationModifiers: {
    default: () => ({}),
    type: Object
  },
  name: {
    type: String,
    default: ''
  },
  nameModifiers: {
    default: () => ({}),
    type: Object
  }
},
[...]

```

Notice that we are not declaring any particular default state for the `capitalize` property of these objects. We simply declare that each of these properties will default to an empty object.

Modifiers will be added to these props as booleans, which means that if no modifiers are received by the component instance, they simply will remain an empty object.

If a modifier like `capitalize` is added, the object will reflect it by adding a `true` value to the respective modifier that was added.

```jsx
{
  capitalize: true
}

```

With this knowledge in mind, we are going to refactor our component. Let’s first move the `$emit` declarations out of the `template`, and make functions that will hold all of our logic.

**📃SalutationName.vue**

```html
<template>
  <div>
    <select
      name="salutation"
      @change="updateSalutation"
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
      @input="updateName"
      type="text"
      name="name"
    />
  </div>
</template>

<script>
const salutations = [...]

export default {
  props: {
    salutation: {
      type: String,
      default: ''
    },
// Holds the modifiers for the salutation v-modelsalutationModifiers: {
      default: () => ({})
    },
    name: {
      type: String,
      default: ''
    },
// Holds the modifiers for the name v-modelnameModifiers: {
      default: () => ({})
    }
  },
  setup (props, { emit }) {
    const updateSalutation = event => {
      let val = event.target.value

      emit('update:salutation', val)
    }

    const updateName = event => {
      let val = event.target.value

      emit('update:name', val)
    }

    return {
      salutations,
      updateSalutation,
      updateName
    }
  }
}
</script>

```

Notice that we updated the `template` to reflect the two new functions — `updateSalutation` and `updateName` that we declared in the `setup` function of our component. For now, it does the exact same thing as before.

Notice also that we have modified the `setup()` function to accept a `props` parameter as a first argument, and `{ emit }` from the second argument.

The second argument is the `context` of the component instance, which in return holds an `emit` property which has the `$emit` function in it. We are using JavaScript deconstructing to extract only the `emit` function directly.

Now that we’re done refactoring, we can add some logic inside our setup `updateX` functions to modify the emitted value in case that a modifier is present. This is where the `props` that we added earlier are going to shine, since now we can use a simple `if` statement to check if they are evaluating to `true` and modify our emitted value.

**📃SalutationName.vue**

```jsx
[...]
setup (props, { emit }) {
  const updateSalutation = event => {
    let val = event.target.value
    if (props.salutationModifiers.capitalize) {
      val = val.toUpperCase()
    }

    emit('update:salutation', val)
  }

  const updateName = event => {
    let val = event.target.value
    if (props.nameModifiers.capitalize) {
      val = val.charAt(0).toUpperCase() + val.slice(1)
    }

    emit('update:name', val)
  }

  return {
    salutations,
    updateSalutation,
    updateName
  }
}
[...]

```

That’s it! Now we can go back to our browser and test our new modifiers in action.

![https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2Fresults.jpg?alt=media&token=8c902a59-f137-4bf7-ab60-8cd0c09f8194](https://firebasestorage.googleapis.com/v0/b/vue-mastery.appspot.com/o/flamelink%2Fmedia%2Fresults.jpg?alt=media&token=8c902a59-f137-4bf7-ab60-8cd0c09f8194)

## **Extra challenge**

Are you up for a little fun? Try implementing an extra modifier for `name` called `reverse` that reverses everything the user types.

Tip: You’ll have to chain the modifiers together in the `v-model` declaration like so:

```jsx
v-model:name.capitalize.reverse="form.name"

```

---

## **Coming up next**

In this lesson, you learned how to create and use your own modifiers for your `v-model` ready components.

In our next lesson, we are going to take a deep dive into `$attrs`, and some of the key differences between Vue 2 and Vue 3 regarding attributes like `class` and`style`.

We’ll also take a look at the disappearing act of `$listeners` and what that means for us developers in terms of component composition.

See you in the next lesson!