<template>
    <div class="q-gutter-md row items-center">
      <q-spinner-cube
        v-for="size in ['xs', 'sm', 'md', 'lg', 'xl']"
        :key="size"
        :size="size"
        color="primary"
      />
    </div>
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'SpinnerFun'
})
</script>
