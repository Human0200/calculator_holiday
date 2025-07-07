<!-- src/components/Sidebar.vue -->
<template>
  <div>
    <sidebar-menu 
      :menu="menu"
      :collapsed="collapsed"
      :theme="selectedTheme"
      :show-one-child="true"
      @toggle-collapse="onToggleCollapse"
      @item-click="onItemClick"
    >
      <template #header>
        <div class="logo-container" @click="goToHome">
          <img :src="logo" alt="Company Logo" class="logo">
        </div>
      </template>
    </sidebar-menu>
  </div>
</template>

<script>
import { SidebarMenu } from 'vue-sidebar-menu'
import 'vue-sidebar-menu/dist/vue-sidebar-menu.css'

export default {
  name: 'AppSidebar',
  components: {
    SidebarMenu
  },
  props: {
    logo: {
      type: String,
      default: require('@/assets/logo.png')
    }
  },
  data() {
    return {
      collapsed: false,
      selectedTheme: 'white-theme',
      menu: [
        {
          header: 'Навигация',
          hiddenOnCollapse: true
        },
        {
          href: '/',
          title: 'Главная',
          icon: 'fa fa-home'
        },
        {
          href: '/about',
          title: 'О компании',
          icon: 'fa fa-info-circle',
          child: [
            {
              href: '/about/history',
              title: 'История'
            },
            {
              href: '/about/team',
              title: 'Команда'
            }
          ]
        },
        {
          href: 'https://company.com',
          title: 'Наш сайт',
          icon: 'fa fa-globe',
          target: '_blank'
        },
        {
          header: 'Документы',
          hiddenOnCollapse: true
        },
        {
          title: 'Политики',
          icon: 'fa fa-file-text',
          child: [
            {
              href: '/docs/privacy',
              title: 'Конфиденциальность'
            },
            {
              href: '/docs/terms',
              title: 'Условия'
            }
          ]
        }
      ]
    }
  },
  methods: {
    onToggleCollapse(collapsed) {
      this.collapsed = collapsed
      this.$emit('toggle-collapse', collapsed)
    },
    onItemClick(event, item) {
      this.$emit('item-click', event, item)
    },
    goToHome() {
      this.$emit('go-home')
    }
  }
}
</script>

<style scoped>
.logo-container {
  padding: 20px;
  text-align: center;
  cursor: pointer;
}

.logo {
  max-width: 80%;
  height: auto;
  transition: all 0.3s;
}

.logo:hover {
  transform: scale(1.05);
}

.v-sidebar-menu {
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}

@media (max-width: 768px) {
  .logo {
    width: 30px;
  }
  
  .v-sidebar-menu {
    width: 50px !important;
  }
  
  .v-sidebar-menu.vsm_collapsed {
    width: 50px !important;
  }
}
</style>