<template>
  <aside class="site-sidebar">
    <div class="site-sidebar__inner">
      <el-menu
        :default-active="menuNavActive"
        :collapse="$store.state.sidebarCollapse"
        class="site-sidebar__menu"
        background-color="#545c64"
        text-color="#fff"
        active-text-color="#ffd04b">
        <el-menu-item index="1-1" @click="$router.push({ name: 'home' })">
          <icon-svg name="shouye" class="site-sidebar__menu-icon"></icon-svg>
          <span slot="title">首页</span>
        </el-menu-item>
        <sub-menu-nav
          v-for="menuNav in $store.state.menuNavList" 
          :key="menuNav.menuId"
          :menu-nav="menuNav">
        </sub-menu-nav>
      </el-menu>
    </div>
  </aside>
</template>

<script>
  import SubMenuNav from '@/components/sub-menu-nav'
  import API from '@/api'
  import { mapMutations } from 'vuex'
  import { getRouteNameByUrl } from '@/utils'
  import isEmpty from 'lodash/isEmpty'
  export default {
    data () {
      return {
        menuNavActive: '1-1'
      }
    },
    components: {
      SubMenuNav
    },
    watch: {
      $route: 'routeHandle'
    },
    created () {
      this.getMenuNavList().then(() => {
        this.routeHandle(this.$route)
      })
    },
    methods: {
      // 获取菜单导航列表 / 权限
      getMenuNavList () {
        return API.menu.nav().then(({data}) => {
          if (data && data.code === 0) {
            this.UPDATE_MENU_NAV_LIST(data.menuList)
            sessionStorage.setItem('permissions', JSON.stringify(data.permissions || '[]'))
          } else {
            this.UPDATE_MENU_NAV_LIST([])
            sessionStorage.setItem('permissions', '[]')
          }
        })
      },
      // 路由操作
      routeHandle (route) {
        if (/^\/n\/.*$/.test(route.path)) {
          var tab = this.$store.state.contentTabs.filter(item => item.name === route.name)
          // tab不存在, 先添加
          if (isEmpty(tab)) {
            var menuNav = this.getMenuNavByRouteName(route.name, this.$store.state.menuNavList)
            if (!isEmpty(menuNav)) {
              this.menuNavActive = menuNav.menuId + ''
              this.ADD_CONTENT_TAB({
                id: menuNav.menuId,
                name: route.name,
                title: menuNav.name,
                type: (window.SITE_CONFIG.nestIframeRouteNameList || []).indexOf(route.name) !== -1 ? 'iframe' : 'module',
                url: menuNav.url
              })
            }
          }
          this.UPDATE_CONTENT_TABS_ACTIVE_NAME({ name: route.name })
        }
      },
      // 获取菜单导航, 根据路由名称
      getMenuNavByRouteName (name, menuNavList) {
        for (var i = 0; i < menuNavList.length; i++) {
          if (menuNavList[i].list && menuNavList[i].list.length >= 1) {
            return this.getMenuNavByRouteName(name, menuNavList[i].list)
          } else {
            if (getRouteNameByUrl(menuNavList[i].url) === name) {
              return menuNavList[i]
            }
          }
        }
      },
      ...mapMutations(['UPDATE_MENU_NAV_LIST', 'ADD_CONTENT_TAB', 'UPDATE_CONTENT_TABS_ACTIVE_NAME'])
    }
  }
</script>
