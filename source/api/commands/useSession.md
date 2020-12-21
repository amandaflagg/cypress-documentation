---
title: useSession
containerClass: experimental
---

{% note warning %}
{% fa fa-warning orange %} **This is an experimental feature. In order to use it, you must set the {% url "`experimentalSessionSupport`" experiments %} configuration option to `true`.**
{% endnote %}

Use `cy.useSession()` to set client-stored persisted data in your application. A Session consists of `cookies` and `localStorage` created when using the accompanying {% url `cy.defineSession()` defineSession %} command.

With `cy.useSession()`, you can...

* set `cookies` and `localStorage` without use of `cy.request()` or `cy.visit()` before every test.
* switch between multiple `Sessions` during a single test.
* inspect Session data that applied to test in the Command Log.
* quickly iterate as an authenticated user in during `cypress open` without replaying login steps.
* keep the convenience and test coverage of UI based authentication while gaining the speed of programmatic login.
* reduce your spec run time. Since Session data is saved after the first time it's used, future tests that share a call to `cy.useSession()` will use Session restoration without needing to perform slower UI actions.

{% note warning %}
**Note:** when using Sessions, all cookies and localStorage will be cleared before every test *for all domains*.
{% endnote %}

# Syntax

## Usage

```ts
cy.useSession(sessionReference)
```

## Arguments

### **{% fa fa-angle-right %} sessionReference** **_(`string | object`)_**

Specifies the Session reference to apply, as previously defined in {% url `cy.defineSession()` defineSession %}. This is either the name (`string`) of a Session or the return value of the {% url `cy.defineSession()` defineSession %} command.

```ts
cy.useSession('mySession')

// OR

const mySession = cy.defineSession({
  name: 'mySession'
  steps () {...}
})

cy.useSession(mySession)
```

# Examples

## Session applied in a `beforeEach` hook

```ts
// can be placed inside spec or in a cypress/support file
cy.defineSession({
  name: 'userLoggedIn',
  steps () {
    cy.visit('/login')
    cy.get('.username').type('janelane')
    cy.get('.password').type('secret1')
    cy.get('.login').click()
  }
})

describe('as a logged in user', ()=>{
  beforeEach(()=>{
    cy.useSession('userLoggedIn')
    // useSession() always navigates you to a blank page
    cy.visit('/profile')
  })

  it('correct user shown in profile', ()=>{
    // beforeEach executed session steps
    // and saved the 'userLoggedIn' Session 
    cy.get('.profile-name').should('contain', 'Jane')
  })

  it('user can log out', ()=>{
    // beforeEach did NOT execute session steps
    // and instead restored saved 'userLoggedIn' session data
    cy.get('.logout-btn').click()
  })
})
```

## Switch users within a single test

```ts
it('admin can ban user', ()=> {
  cy.useSession('admin')
  cy.visit('/users/nefarious')
  cy.get('.ban-user-btn').click()
  // wait for the network request to be completed
  cy.get('.toast').contains('successfully banned user')

  cy.useSession('nefariousUser')
  cy.visit('/dashboard')
  cy.contains('sorry, you can no longer access this site')
})
```

# See also

* {% url `cy.defineSession()` defineSession %}
