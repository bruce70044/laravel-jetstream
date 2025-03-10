<template>
  <div
    :class="containerClass"
    :data-theme="colorScheme"
    @click="onDocumentClick($event)"
  >
    <div class="layout-content-wrapper">
      <AppTopBar
        :topbarNotificationMenuActive="topbarNotificationMenuActive"
        :topbarUserMenuActive="topbarUserMenuActive"
        @menu-button-click="onMenuButtonClick"
        @search-click="toggleSearch"
        @topbar-notification="onTopbarNotificationMenuButtonClick"
        @topbar-user-menu="onTopbarUserMenuButtonClick"
        @right-menu-click="onRightMenuButtonClick"
        @right-menubutton-click="onRightMenuButtonClick"
      ></AppTopBar>

      <div class="layout-content">
        <slot></slot>
      </div>
      <AppFooter />
    </div>

    <AppMenu
      :model="menu"
      :layoutMode="layoutMode"
      :active="menuActive"
      :mobileMenuActive="staticMenuMobileActive"
    ></AppMenu>

    <AppRightMenu
      :rightMenuActive="rightMenuActive"
      @right-menu-click="onRightMenuClick"
    ></AppRightMenu>

    <AppConfig
      v-model:configActive="configActive"
      v-model:layoutMode="layoutMode"
      v-model:menuTheme="menuTheme"
      v-model:colorScheme="colorScheme"
      @config-click="onConfigClick"
      @config-button-click="onConfigButtonClick"
    ></AppConfig>

    <AppSearch
      :searchActive="searchActive"
      @search-click="onSearchClick"
      @search-hide="onSearchHide"
    />

    <div class="layout-mask modal-in"></div>
  </div>
</template>

<script>
import EventBus from "../event-bus";
import AppTopBar from "./AppTopbar";
import AppFooter from "./AppFooter";
import AppConfig from "./AppConfig";
import AppMenu from "./AppMenu";
import AppSearch from "./AppSearch";
import AppRightMenu from "./AppRightMenu";



export default {
  data() {
    return {
      menuActive: false,
      layoutMode: "static",
      colorScheme: "light",
      menuTheme: "layout-sidebar-darkgray",
      overlayMenuActive: false,
      staticMenuDesktopInactive: false,
      staticMenuMobileActive: false,
      menuClick: false,
      searchActive: false,
      searchClick: false,
      userMenuClick: false,
      topbarUserMenuActive: false,
      notificationMenuClick: false,
      topbarNotificationMenuActive: false,
      rightMenuClick: false,
      rightMenuActive: false,
      configActive: false,
      configClick: false,
      menu: [
        {
          label: "Backoffice",
          icon: "pi pi-fw pi-home",
          items: [
              {
                  label: "Dashboard",
                  icon: "pi pi-fw pi-home",
                  to: "/dashboard"
              },
              {
                  label: "Pedidos",
                  icon: "pi pi-fw pi-shopping-cart",
                  to: "/pedidos",
              },
              {
                  label: "Financeiro",
                  icon: "pi pi-fw pi-money-bill",
                  to: "/financeiros",
              },
              // {
              //     label: "Suporte",
              //     icon: "pi pi-fw pi-comments",
              //     to: "/clientes",
              // },

          ],
        },
        { separator: true },
        {
          label: "Cadastro",
          icon: "pi pi-fw pi-id-card",
          items: [
            {
              label: "Produtos",
              icon: "pi pi-fw pi-table",
              to: "/produtos",
            },

              {
                  label: "Clientes",
                  icon: "pi pi-fw pi-id-card",
                  to: "/clientes",
              },
              {
                  label: "Condições",
                  icon: "pi pi-fw pi-credit-card",
                  to: "/condpgtos",
              },
              {
                  label: "Formas de Pagamento",
                  icon: "pi pi-fw pi-wallet",
                  to: "/formapgtos",
              },
              {
                  label: "Tipos de Movimento",
                  icon: "pi pi-fw pi-sitemap",
                  to: "/tipomovs",
              },
              {
                  label: "Tipos de Frete",
                  icon: "pi pi-fw pi-external-link",
                  to: "/tipofretes",
              },
          ],
        },
        { separator: true },
        {
          label: "Visitação",
          icon: "pi pi-fw pi-desktop",
          items: [
            {
                label: "Fluxo de Visitas",
                icon: "pi pi-fw pi-desktop",
                to: "/display" },
            {
              label: "Pesquisas",
              icon: "pi pi-fw pi-search",
              to: "/elevation",
            },
          ],
        },
        { separator: true },
        {
          label: "Administrativo",
          icon: "pi pi-fw pi-pencil",
          items: [
            {
                label: "Usuários",
                icon: "pi pi-fw pi-user",
                to: "/users" },
            {
              label: "Calendário",
              icon: "pi pi-fw pi-calendar-plus",
              to: "/calendario",
            },
            {
              label: "Log de Atividade",
              icon: "pi pi-fw pi-lock",
              to: "/activitylogs",
            },

          ],
        },
      ],
    };
  },
  computed: {
    containerClass() {
      return [
        "layout-wrapper",
        {
          "layout-overlay": this.layoutMode === "overlay",
          "layout-static": this.layoutMode === "static",
          "layout-slim": this.layoutMode === "slim",
          "layout-sidebar-dim": this.colorScheme === "dim",
          "layout-sidebar-dark": this.colorScheme === "dark",
          "layout-overlay-active": this.overlayMenuActive,
          "layout-mobile-active": this.staticMenuMobileActive,
          "layout-static-inactive":
            this.staticMenuDesktopInactive && this.layoutMode === "static",
          "p-input-filled": this.$appState.inputStyle === "filled",
          "p-ripple-disabled": !this.$primevue.config.ripple,
        },
        this.colorScheme === "light" ? this.menuTheme : "",
      ];
    },
  },
  components: {
    AppTopBar,
    AppFooter,
    AppConfig,
    AppMenu,
    AppSearch,
    AppRightMenu,
    //Produtos,
  },
  watch: {
    $route() {
      this.menuActive = false;
      this.$toast.removeAllGroups();
    },
  },
  methods: {
    onDocumentClick() {
      if (!this.searchClick && this.searchActive) {
        this.onSearchHide();
      }

      if (!this.userMenuClick) {
        this.topbarUserMenuActive = false;
      }

      if (!this.notificationMenuClick) {
        this.topbarNotificationMenuActive = false;
      }

      if (!this.rightMenuClick) {
        this.rightMenuActive = false;
      }

      if (!this.menuClick) {
        if (this.isSlim()) {
          EventBus.emit("reset-active-index");
          this.menuActive = false;
        }

        if (this.overlayMenuActive || this.staticMenuMobileActive) {
          this.hideOverlayMenu();
        }

        this.unblockBodyScroll();
      }

      if (this.configActive && !this.configClick) {
        this.configActive = false;
      }

      this.searchClick = false;
      this.configClick = false;
      this.userMenuClick = false;
      this.rightMenuClick = false;
      this.notificationMenuClick = false;
      this.menuClick = false;
    },
    onMenuClick() {
      this.menuClick = true;
    },
    onMenuButtonClick(event) {
      this.menuClick = true;
      this.topbarUserMenuActive = false;
      this.topbarNotificationMenuActive = false;
      this.rightMenuActive = false;

      if (this.isOverlay()) {
        this.overlayMenuActive = !this.overlayMenuActive;
      }

      if (this.isDesktop()) {
        this.staticMenuDesktopInactive = !this.staticMenuDesktopInactive;
      } else {
        this.staticMenuMobileActive = !this.staticMenuMobileActive;
        if (this.staticMenuMobileActive) {
          this.blockBodyScroll();
        } else {
          this.unblockBodyScroll();
        }
      }

      event.preventDefault();
    },
    onMenuItemClick(event) {
      if (!event.item.items) {
        EventBus.emit("reset-active-index");
        this.hideOverlayMenu();
      }
      if (!event.item.items && this.isSlim()) {
        this.menuActive = false;
      }
    },
    onRootMenuItemClick() {
      this.menuActive = !this.menuActive;
    },
    onTopbarUserMenuButtonClick(event) {
      this.userMenuClick = true;
      this.topbarUserMenuActive = !this.topbarUserMenuActive;

      this.hideOverlayMenu();

      event.preventDefault();
    },
    onTopbarNotificationMenuButtonClick(event) {
      this.notificationMenuClick = true;
      this.topbarNotificationMenuActive = !this.topbarNotificationMenuActive;

      this.hideOverlayMenu();

      event.preventDefault();
    },
    toggleSearch() {
      this.searchActive = !this.searchActive;
      this.searchClick = true;
    },
    onSearchClick() {
      this.searchClick = true;
    },
    onSearchHide() {
      this.searchActive = false;
      this.searchClick = false;
    },
    onRightMenuClick() {
      this.rightMenuClick = true;
    },
    onRightMenuButtonClick(event) {
      this.rightMenuClick = true;
      this.rightMenuActive = !this.rightMenuActive;
      this.hideOverlayMenu();
      event.preventDefault();
    },
    onConfigClick() {
      this.configClick = true;
    },
    onConfigButtonClick() {
      this.configActive = !this.configActive;
      this.configClick = true;
    },
    hideOverlayMenu() {
      this.overlayMenuActive = false;
      this.staticMenuMobileActive = false;
      this.unblockBodyScroll();
    },
    blockBodyScroll() {
      if (document.body.classList) {
        document.body.classList.add("blocked-scroll");
      } else {
        document.body.className += " blocked-scroll";
      }
    },
    unblockBodyScroll() {
      if (document.body.classList) {
        document.body.classList.remove("blocked-scroll");
      } else {
        document.body.className = document.body.className.replace(
          new RegExp(
            "(^|\\b)" + "blocked-scroll".split(" ").join("|") + "(\\b|$)",
            "gi"
          ),
          " "
        );
      }
    },
    isSlim() {
      return this.layoutMode === "slim";
    },
    isOverlay() {
      return this.layoutMode === "overlay";
    },
    isDesktop() {
      return window.innerWidth > 991;
    },
  },
};
</script>

<style lang="scss">
</style>
