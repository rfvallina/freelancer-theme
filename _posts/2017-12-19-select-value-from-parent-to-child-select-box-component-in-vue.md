---
layout: post
title: "Select a value from parent to child component in Vue"
date: 2017-12-19 00:00:00 +0200
comments: true
category: blog
tags: [javascript, vue, vuetify]
---

When you land on [Vue](https://vuejs.org/) it's quite common to go a step further and choose a UI component framework to feed your app with a nice aspect from the very beginning and be able to use out-of-the-box components. There are plenty of them. Personally, I chose [Vuetify](https://vuetifyjs.com/). Vuetify follows [Material Design specs](https://material.io/).
<!-- Read More -->

In this post I want to show how you can indicate from a parent component what element must be selected on a select box ```<v-select>``` element which is part of a child component. It's not complicated at all but it's a bit tricky and took me while to figure it out.

![diagram](/img/blog/parent_child_component.svg)


<p align="center">
<img src="/img/blog/vue_folder_structure.png">
</p>


The key is to use ```updated``` lifecycle hook combined with an auxiliar prop in the subcomponent.
This method will be invoked every time the prop gets updated from the parent component. In our case we use a prop called ```selected```. When we get an update on the subcomponent, we update the model property ```sel``` which controls the id currently selected.

***Printers.vue***

```html 
<template>
  <v-select
      label="Printer"
      v-model="sel"
      :items="list"
      item-value="id"
      item-text="name"
      @input="changePrinter()"
      required
    ></v-select>
</template>

<script>
  export default {
    data: function(){
      return {
        sel: 0
      }
    },
    updated: function(){
      this.sel = this.selected;
    },
    props: ['list','selected'],
    methods: {
      changePrinter: function(){
        console.log("child changed: " + this.sel);
        this.$emit('change',this.sel);
      }
    }
  }
</script>
```


***PrinterControl.vue***

```html
<template>
  <div>
    <h2>Printers</h2>
    <printers :list="printers" :selected="selected" @change="printerChanged"></printers>
  </div>
</template>

<script>
import Printers from './Printers.vue';

export default {
  data() {
    return {
      printers: [
        {
          name: 'DYMO LabelWriter 450 Turbo',
          id: 1
        },
        {
          name: 'Zebra UPS 2844',
          id: 2
        },
        {
          name: 'CAB MACH4/300',
          id: 3
        }
      ],
      selected: 1
    }
  },
  mounted (){
    let vm = this;
    setInterval(function(){
      console.log("Selecting a different value");
      console.log(vm.printers);
      vm.selected = (vm.selected == vm.printers.length) ? 1 : vm.selected + 1;
    }, 2000);
  },
  methods: {
    printerChanged: function(){
      console.log("Printer changed. Selected ID: ", this.selected);
    }
  },
  components: {
    Printers
  }
}

</script>
```

To illustrate this behaviour I created a function wrapped in a ```setInterval``` method which updates the property every 2 seconds.
[See it in action](http://demo.rfvallina.com/combo-selector/)

You can [download the source code from github](https://github.com/rfvallina/combo-selector)
