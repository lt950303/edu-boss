<template>
  <div>
    <role-list />
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import RoleList from '@/views/role/components/List.vue'

export default Vue.extend({
  name: 'Role',
  components: {
    RoleList
  }
})
</script>

<style lang="scss" scoped>

</style>
