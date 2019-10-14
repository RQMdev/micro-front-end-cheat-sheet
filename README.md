## Blank Sheet

Import an application :

define a reactModule :

```javascript
// config.js
export const reactModule = {
  id: 'react-app',
  url: 'http://localhost:3000',
  scripts: [
    '/static/js/runtime-main.js',
    '/static/js/main.chunk.js',
    '/static/js/2.chunk.js'
  ],
  styles: ['/static/css/main.chunk.css']
}
```

import it in page.js :

```javascript
// page.js
import { reactModule } from '../config'
```

define a page component that will be the anchor for the coming app :
create the root anchor :

```javascript
// connectedCallback()
const frameworkRoot = this.createElementFromID(reactModule)
this.appendChild(frameworkRoot)

// method:
createElementFromID(appModule) {
  const root = document.createElement('div')
  root.id = appModule.id
  return root
}
```

DEMO: the div with ID

create the promiseSerial helper :

```javascript
// page.js
promiseSerial(funcs, init) {
  return funcs.reduce(
    (promise, func) => promise.then(func),
    Promise.resolve(init)
  )
}
```

create the loadScript method :

```javascript
// page.js
loadScripts(scripts) {
  return this.promiseSerial(
    scripts.map(scriptPath => () =>
      new Promise((resolve, reject) => {
        const previousScript = document.getElementById(scriptPath)
        if (previousScript) {
          previousScript.remove()
        }
        const script = document.createElement('script')
        script.src = scriptPath
        script.id = scriptPath
        script.defer = true
        script.type = 'text/javascript'
        script.onload = resolve
        script.onerror = reject
        document.body.appendChild(script)
      })
    )
  )
}
```

add it into the connectedCallback() :

```javascript
// page.js
this.loadScripts(
  reactModule.scripts.map(scriptPath => reactModule.url + scriptPath)
)
```

DEMO: the cat is here ! but not the app style !!

add loadStyles method :

```javascript
// page.js
loadStyles(styles) {
  return this.promiseSerial(
    styles.map(stylePath => () =>
      new Promise((resolve, reject) => {
        const previousStyle = document.getElementById(stylePath)
        if (previousStyle) {
          previousStyle.remove()
        }
        const style = document.createElement('link')
        style.href = stylePath
        style.id = stylePath
        style.rel = 'stylesheet'
        style.onload = resolve
        style.onerror = reject
        document.head.appendChild(style)
      })
    )
  )
}
```

call it from connectedCallback :

```javascript
// router.js
this.loadStyles(
  reactModule.styles.map(stylePath => reactModule.url + stylePath)
)
```

DEMO: The cat is now stylish :D

## Save : react-imported

But i wanna go to the other page of my final application !
let's define our other modules :

```javascript
// config.js
const angularModule = {
  tag: 'app-root',
  url: 'http://localhost:4200',
  scripts: [
    '/main-es2015.js',
    '/polyfills-es2015.js',
    '/runtime-es2015.js',
    '/styles-es2015.js',
    '/vendor-es2015.js'
  ],
  styles: []
}

const vueModule = {
  id: 'vue-app',
  url: 'http://localhost:8080',
  scripts: ['/js/app.js', '/js/chunk-vendors.js'],
  styles: ['/css/app.css']
}
```

let's introduce some route, that we will match with the modules!

```javascript
// config.js
export const routes = [
  {
    path: '/',
    exact: true,
    module: reactModule
  },
  {
    path: '/chien',
    title: 'Chien',
    module: angularModule
  },
  {
    path: '/chat',
    title: 'Chat',
    module: reactModule
  },
  {
    path: '/canard',
    title: 'Canard',
    module: vueModule
  }
]
```

Now that we only export routes: let's refactor our page :

```javascript
// page.js
import { routes } from '../config'

// connectedCallback()
const appModule = routes.find(route =>
  route.exact
    ? route.path === location.pathname
    : location.pathname.startsWith(route.path)
).module
```

!! Rename all reactModule to appModule

DEMO: Now we can go the each URL to see each module !!!
React: Good,
Vue: Good,
Angular ... failed ! 'cause it doesn't anchor to a div with an id ! but to a tag !
let's refactor our createElementFromID method :

```javascript
createElementFromID(appModule) {
    let root = null
    if (appModule.tag) {
      root = document.createElement(appModule.tag)
    } else {
      root = document.createElement('div')
      root.id = appModule.id
    }
    return root
  }
```

## Save all-application-on-board

Let's make a navigation components !

```javascript
import { routes } from '../config'

export default class ContainerNavbar extends HTMLElement {
  constructor() {
    super()
  }

  static get tagName() {
    return 'container-navbar'
  }

  render() {
    this.innerHTML = `
    <nav role="navigation">
      <ul>
      ${routes
        .filter(route => route.title)
        .map(
          route => `
            <li>
              <container-link
                path="${route.path}"
                title="${route.title}"
              ></container-link>
            </li>
          `
        )
        .join('')}
      </ul>
    </nav>
  `
  }

  connectedCallback() {
    this.render()
  }
}
```

b) and the sub component link :

```javascript
export default class ContainerLink extends HTMLElement {
  constructor() {
    super()
  }

  static get tagName() {
    return 'container-link'
  }

  connectedCallback() {
    this.path = this.getAttribute('path')
    this.title = this.getAttribute('title')

    this.render()
  }

  render() {
    this.innerHTML = `
      <a href="${this.path}">
        ${this.title}
      </a>
    `
  }
}
```

c) and import them in component/index.js

```javascript
export { default as ContainerLink } from './link'
export { default as ContainerNavbar } from './navbar'
```

d) place them in the app component :

```javascript
  <container-navbar></container-navbar>
  <div id="app-container">

  </div>
```

## Save navigation-done

And TADA !! We are done ! we got a complete app where we can navigate to different pages that are from different framework and that are exposed in different place...

but...

The page reloads !!!

Let's add a router to handle that and not reload all the html :

first add router component to our app :

```javascript
// app.js
connectedCallback() {
  this.innerHTML = `
    <container-router>
      <container-navbar></container-navbar>
      <div id="app-container">
        <router-outlet></router-outlet>
      </div>
    </container-router>
  `
}
```

then create the router comp :

```javascript
export default class ContainerRouter extends HTMLElement {
  constructor() {
    super()
  }

  static get tagName() {
    return 'container-router'
  }

  connectedCallback() {
     this.addEventListener('click', event => {
      event.preventDefault()
     }
  }
}
```

export it in component/index.js

```javascript
export { default as ContainerRouter } from './router'
```

create the resolveRoute method:

```javascript
// router.js
import { routes } from '../config'

// connectedCallback()
  this.resolveRoute(location.pathname)

// resolveRoute
resolveRoute(path) {
        console.log('resolveRoute with path :', path)
        const targetRoute = this.findRoute(path)
        console.log('targetRoute', targetRoute)
        if (!targetRoute) {
          this.resolveRoute('/')
        }

   // render Route here
  }

  findRoute(path) {
    return routes.find(route =>
      route.exact ? route.path === path : path.startsWith(route.path)
    )
  }
```

DEMO: Console log finding the correct path and finding the targeted Route

```javascript
//router.js

// resolveRoute
this.renderRoute()

// renderRoute
renderRoute() {
  const routerOutlet = this.querySelector('router-outlet')
  routerOutlet.innerHTML = '<container-page></container-page>'
}
```

DEMO: We have our module back ! But the link still don't work ;)

Let make our click listener do the job:

```javascript
//router.js
// click listener
  const href = this.getLinkHref(event.target)
  if (href) {
    const { pathname } = new URL(href, location.origin)
    this.resolveRoute(pathname, event)
  }

// getLinkHref
  getLinkHref(element) {
    if (element.tagName === 'A') {
      return element.href || ''
    } else if (element.tagName !== 'BODY') {
      return this.getLinkHref(element.parentElement)
    } else {
      return undefined
    }
  }
```

Since we are not prevenDefaulting here, let's pass the event to the render route to do it there !

```javascript
// resolveRoute
  resolveRoute(path, event = new Event('click')) {
    // ...
    this.renderRoute(event)
  }

// renderRoute
  renderRoute(event) {
    event.preventDefault()
    // ...
  }
```

DEMO: Now we can click and it re-render the page component : But it's always the same module because the url didn't change !
Let's fix that:

```javascript
// resolveRoute
  this.changePage(path)

 // changePage method
  changePage(path) {
    window.history.pushState(window.history.state, null, path)
  }
```

We can now navigate without the page reloading completely (see the log staying there !!)
But we have our page blinking even when we are staying on the same app !

We need a new fix :
We need to compare old module to new one, so let's pass targetRoute to our renderRoute method

```javascript
// resolveRoute:
  this.renderRoute(targetRoute, event)

// renderRoute:
renderRoute(targetRoute, event) {
  event.preventDefault()
  if (this.currentRoute !== targetRoute) {
    const routerOutlet = this.querySelector('router-outlet')
    routerOutlet.innerHTML = '<container-page></container-page>'
    this.currentRoute = targetRoute
  }
}
```

We still have a last problem : the react link not working anymore !
We have to be more precise when event.preventDefault because it's breaking react behaviour
let's pass the path to renderRoute:

```javascript
//resolveRoute
this.renderRoute(targetRoute, event, path)

// renderRoute:
renderRoute(targetRoute, event, path) {
  if (
    this.currentRoute &&
    !targetRoute.path.startsWith(this.currentRoute.path)
  ) {
    event.preventDefault()
  }

  if (this.currentRoute !== targetRoute) {
    const routerOutlet = this.querySelector('router-outlet')
    routerOutlet.innerHTML = '<container-page></container-page>'
    this.currentRoute = targetRoute
    this.currentPath = path
  }
}
```

And that's it ;)
