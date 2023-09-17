<template>
  <header>
    <h1 class="logo">Experimental version!</h1>
    <div class="wrapper">
      <h3>Some explanations to the text editor</h3>
      <ol>
        <li>Type a text you want to customise</li>
        <li>Select text and choose styles you want</li>
        <li>If you select two customized block, IT WON'T WORK correctly</li>
        <li>If you want reset styles for new text, just "enter"</li>
        <li>For parsing, click "parse", it will console.log result</li>
      </ol>
    </div>
  </header>

  <main>
        <TextEditor
            classes="box-shadow"
            :width="700"
            :height="300"
            @update:value="updateValue"
        />

  </main>
</template>
<script>
import TextEditor from './components/TextEditor.vue'

export default {
  data() {
    return {
      value: '',
    }
  },
  methods: {
    updateValue(value) {
      this.value = value;
    },
  },
  components: {
    TextEditor,
  }
}
</script>
<style>
.box-shadow {
  box-shadow: 4px 2px 13px 4px rgba(0,0,0,0.53);
  -webkit-box-shadow: 4px 2px 13px 4px rgba(0,0,0,0.53);
  -moz-box-shadow: 4px 2px 13px 4px rgba(0,0,0,0.53);
}
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    flex-direction: column;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    padding-top: 20px;
  }
}
</style>
