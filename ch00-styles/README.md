# Styles

In a visual testing tool, where the tool can also be used for design, leaving out styles would be a pity. We want our components to look the same in the component tests and the served app.

We have some needs to emphasize component test driven design; we do not want styles to be of importance in the component code and we want styles in the rendered component, whether it is the app being served or a component test. For this reason we will prefer to use global styles, and take advantage of [react-icons](https://react-icons.github.io/react-icons) where profitable.

To setup global styles Add `node-sass` and `bulma` to dependencies.

```bash
yarn add -D node-sass
yarn add bulma
```

Create `src/styles.scss` and paste the following. We will be importing this file in the component tests.

```scss
$vue: #42b883;
$vue-light: #42b883;
$angular: #b52e31;
$angular-light: #eb7a7c;
$react: #00b3e6;
$react-light: #61dafb;
$primary: $react;
$primary-light: $react-light;
$link: $primary; // #00b3e6; // #ff4081;

$shade-light: #fafafa;

@import "bulma/bulma.sass";

.menu-list .active-link,
.menu-list .router-link-active {
  color: #fff;
  background-color: $link;
}

.not-found {
  i {
    font-size: 20px;
    margin-right: 8px;
  }
  .title {
    letter-spacing: 0px;
    font-weight: normal;
    font-size: 24px;
    text-transform: none;
  }
}

header {
  font-weight: bold;
  font-family: Arial;
  span {
    letter-spacing: 0px;
    &.tour {
      color: #fff;
    }
    &.of {
      color: #ccc;
    }
    &.heroes {
      color: $primary-light;
    }
  }
  .navbar-item.nav-home {
    border: 3px solid transparent;
    border-radius: 0%;
    &:hover {
      border-right: 3px solid $primary-light;
      border-left: 3px solid $primary-light;
    }
  }
  .fab {
    font-size: 24px;
    &.js-logo {
      color: $primary-light;
    }
  }
  .buttons {
    i.fab {
      color: #fff;
      margin-left: 20px;
      margin-right: 10px;
      &:hover {
        color: $primary-light;
      }
    }
  }
}

.edit-detail {
  .input[readonly] {
    background-color: $shade-light;
  }
}

.content-title-group {
  margin-bottom: 16px;
  h2 {
    border-left: 16px solid $primary;
    border-bottom: 2px solid $primary;
    padding-left: 8px;
    padding-right: 16px;
    display: inline-block;
    text-transform: uppercase;
    color: #555;
    letter-spacing: 0px;
    &:hover {
      color: $link;
    }
  }
  button.button {
    border: 0;
    color: #999;
    &:hover {
      color: $link;
    }
  }
}
ul.list {
  box-shadow: none;
}
div.card-content {
  background-color: $shade-light;
  .name {
    font-size: 28px;
    color: #000;
  }
  .description {
    font-size: 20px;
    color: #999;
  }
  background-color: $shade-light;
}
.card {
  margin-bottom: 2em;
}

label.label {
  font-weight: normal;
}

p.card-header-title {
  background-color: $primary;
  text-transform: uppercase;
  letter-spacing: 4px;
  color: #fff;
  display: block;
  padding-left: 24px;
}
.card-footer button {
  font-size: 16px;
  i {
    margin-right: 10px;
  }
  color: #888;
  &:hover {
    color: $link;
  }
}

.modal-card-foot button {
  display: inline-block;
  width: 80px;
}

.modal-card-head,
.modal-card-body {
  text-align: center;
}

.field {
  margin-bottom: 0.75rem;
}

.navbar-burger {
  margin-left: auto;
}

button.link {
  background: none;
  border: none;
  cursor: pointer;
}

.page-loading {
  font-size: 300%;
  position: fixed;
  z-index: 999;
  overflow: show;
  margin: auto;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  width: 50px;
  height: 50px;
}

.icon-loading {
  animation: circle 1.2s steps(8) infinite;
}

@keyframes circle {
  from {
    transform: rotate(90deg);
  }

  to {
    transform: rotate(450deg);
  }
}
```