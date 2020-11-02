<template>
  <button type="button" @click="$emit('handler', text)">{{text}}</button>
</template>

<script>
export default {
  props: ['text']
}
</script>
