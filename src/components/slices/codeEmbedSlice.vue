<template>
  <div class='post-part single container'>
    <prismic-rich-text class="codeEmbedSlice" :field="slice.primary.code_embed"/>
  </div>
</template>

<script>
export default {
  props: ['slice'],
  name: 'code-embed-slice'
}
</script>
