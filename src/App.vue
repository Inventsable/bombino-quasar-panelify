<template>
  <div id="q-app">
    <Panel>
      <!-- <render-dialog /> -->
      <Menus refresh debug />
      <router-view style />
    </Panel>
  </div>
</template>

<script>
// Determine if behavior should adhere to browser or CEP standards (panelify compatibility)
let isBrowser = !window.__adobe_cep__;
// Dynamic CSS variables that automatically handle all app themes and changes:
// https://github.com/Inventsable/starlette
import starlette from "starlette";

// Utility components, see here:
// https://github.com/Inventsable/lokney
import { Menus, Panel } from "lokney";
/*
  Panel component above also includes:
    - Starlette UI theme and color library: 
      https://github.com/Inventsable/starlette
    - CEP-Spy identification and app utility:
      https://github.com/Inventsable/cep-spy
 These are still installed into this panel and can be used when needed like so:
 import spy from 'cep-spy'

 NOTES: 
  - Starlette is already active in your panel! There's no need to initialize it.
  - Need CSInterface or a script? You can use the script-path attribute of Panel to launch scripts or utilities:
    https://github.com/Inventsable/lokney/tree/master/components/Panel
*/

// Dynamic identification object that reports all panel and host information:
// https://github.com/Inventsable/CEP-Spy
const spy = !isBrowser
  ? require("cep-spy").default
  : // Unless for panelify, in which case provide fake data
    require("./utils/fakeSpy").default;

import { mapActions, mapGetters } from "vuex";
import { Dialog, Loading, Notify, LoadingBar } from "quasar";
import showErrorMessage from "src/functions/function-show-error-message.js";

export default {
  name: "App",
  components: {
    Menus,
    "render-dialog": require("src/components/panel/RenderDialog.vue").default
  },
  data: () => ({
    isMounted: false,
    isBrowser: false,
    loading: false
  }),
  computed: {
    ...mapGetters("settings", ["settings", "config"]),
    routePath() {
      return this.$route.path;
    },
    spy() {
      return spy;
    }
  },
  watch: {
    loading(state) {
      return state ? Loading.show() : Loading.hide();
    },
    routePath(val) {
      this.setLastMainRoute(val);
    }
  },
  async mounted() {
    this.loading = false;
    //
    this.getSettings();
    // this.deleteSettings();
    console.log("SETTINGS:", this.settings);
    this.$q.notify.setDefaults({
      position: "top",
      timeout: 0,
      actions: [{ icon: "close", color: "white" }]
    });
  },
  methods: {
    ...mapActions("settings", [
      "getSettings",
      "resetSettings",
      "deleteSettings",
      "setLastMainRoute"
    ]),
    reset() {
      this.loading = true;
      this.deleteSettings();
      setTimeout(() => {
        this.resetSettings();
        setTimeout(() => {
          this.loading = false;
        }, 200);
      }, 400);
    },
    notify(text) {
      this.$q.notify(text);
    },
    error(text) {
      this.loading = false;
      showErrorMessage(text);
    },
    getCSS(prop) {
      return window
        .getComputedStyle(document.documentElement)
        .getPropertyValue(`${/^\-\-/.test(prop) ? prop : "--" + prop}`);
    },
    setCSS(prop, data) {
      document.documentElement.style.setProperty(
        `${/^\-\-/.test(prop) ? prop : "--" + prop}`,
        data
      );
    }
  }
};
</script>

<style>
:root {
  --color-shaded: rgba(0, 0, 0, 0.125);
  --toolbar-height: 48px;
  --bottombar-height: 30px;
  --min-width: 400px;
  --app-scrollbar-width: 16px;
  min-width: var(--min-width);
}

.q-card.q-dialog-plugin {
  background-color: var(--color-bg);
  color: var(--color-icon);
}
.q-drawer {
  background: var(--color-header);
}

.q-item__label--caption {
  color: var(--color-default);
}

.q-item__label--header {
  color: var(--color-text-label);
}

.q-layout__section--marginal {
  color: var(--color-default);
}
.quasar-logo-text {
  fill: var(--color-default);
}
.q-notification {
  background-color: var(--color-header);
}

.q-menu {
  background-color: var(--color-header);
  color: var(--color-default);
}

.q-field__native,
.q-field__prefix,
.q-field__suffix,
.q-field__marginal,
/* .q-field__control, */
.q-field__label,
.q-field__bottom {
  color: var(--color-default);
}

.q-checkbox__inner {
  color: var(--color-default);
}

.q-checkbox__inner--active {
  color: var(--q-color-primary);
}

.q-field__focused {
  border-bottom-color: red;
}

.q-field--standard .q-field__control:before {
  border-bottom: 1px solid var(--color-default);
}
.q-field--standard .q-field__control:hover:before {
  border-bottom: 1px solid var(--color-btn-hover);
}

.q-dark {
  background: var(--color-bg);
}

.q-item__label--header {
  user-select: none;
  cursor: default;
  font-size: 1rem;
}

.q-separator {
  background: var(--color-btn-disabled-text);
}

.q-field__native,
.q-field__prefix,
.q-field__suffix {
  color: var(--color-default);
}

.q-field__marginal {
  color: var(--color-default);
}
</style>
