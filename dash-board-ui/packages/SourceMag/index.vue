<template>
  <div class="db-manage-main-wrap">
    <side-menu @getPageInfo="getPageInfo" />
    <menu-content
      :catalog-info="catalogInfo"
      :type="type"
    />
  </div>
</template>
<script>
import SideMenu from './SideMenu'
import MenuContent from './MenuContent'
export default {
  name: '',
  props: {
    type: {
      type: String,
      default: 'dashboard'
    }
  },
  components: { SideMenu, MenuContent },
  data () {
    return {
      catalogInfo: {
        isAll: true,
        page: {
          id: null,
          code: null,
          type: ''
        }
      }
    }
  },
  mounted () {},
  methods: {
    getPageInfo (pageInfo) {
      this.catalogInfo = pageInfo
    }
  }
}
</script>

<style lang="scss" scoped>
.db-manage-main-wrap {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  background:var(--db-el-background-2) ;
}
</style>
