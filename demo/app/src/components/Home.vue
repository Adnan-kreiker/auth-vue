<template>
  <div id="home">
    <h1 class="ui header">Home Page</h1>
    <div v-if="!user || !user.profile">
      <p>If you‘re viewing this page then you have successfully started this Vue application.</p>
      <p>This example shows you how to use the
        <a href="https://github.com/tiwater/auth-vue">Auth Vue Library</a> to add the OAuth 2.0 to your application.</p>
      <p>When you click the login button below, you will be redirected to the login page. After you authenticate,
        you will be redirected to this application with an ID token and access token. These tokens will be stored in local storage
        and can be retrieved later.</p>
      <button
        id="login-button"
        class="ui primary button"
        role="button"
        v-on:click="login()"
      >
      Login
      </button>
    </div>

    <div v-if="user && user.profile">
      <p>Welcome back, {{user && user.profile.preferred_username}}!</p>
      <p>
        You have successfully authenticated, and have been redirected back to this application.  You now have an ID token and access token in local storage.
        Visit the <a href="/profile">My Profile</a> page to take a look inside the ID token.
      </p>
      <h3>Next Steps</h3>
      <p>
        Currently this application is a stand-alone front end application.
        At this point you can use the access token to authenticate yourself against resource servers that you control.
      </p>
      <p>
        This sample is designed to work with one of our resource server examples.
        To see access token authentication in action, please download one of these resource server examples:
      </p>
      <ul>
        <li
          v-for="(example, index) in resourceServerExamples"
          :key="index"
        >
          <a :href="example.url">{{example.label}}</a>
        </li>
      </ul>
      <p>Once you have downloaded and started the example resource server, you can visit the <a href="/messages">My Messages</a> page to see the authentication process in action.</p>
      <button
        id="logout-button"
        class="ui primary button"
        role="button"
        v-on:click="logout()"
      >
      Logout
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { User, UserManager } from "oidc-client";
import { ref, defineComponent, inject, onMounted } from "vue";

export default defineComponent ({
  name: 'home',
  setup() {
    const userMgr = inject<UserManager>('userMgr')
    const user = ref<User>({} as User);

    const resourceServerExamples = [
      {
        label: 'Resource Server Example',
        url: 'https://github.com/tiwater/auth-vue/tree/main/demo/resource-server'
      }
    ];

    // Force the page refresh after login/logout
    if (userMgr) {
      userMgr.events.addUserLoaded((_user) => {
        console.log('userMgr.events.addUserLoaded');
        user.value = _user;
      })
      userMgr.events.addUserUnloaded(() => {
        console.log('userMgr.events.addUserUnloaded');
        user.value = {} as User;
      })
    }

    onMounted(() => { 
      userMgr!.getUser().then((_user) => {
        if (_user) {
          user.value = _user;
      }}); 
    });

    function login () {
      console.log('Home: login');
      userMgr!.signinRedirect({state: window.location.pathname }).then(user => {
        console.log('Home userMgr.signinRedirect:', user);
      }).catch((err: any) => { console.log('Login Error: ', err); });
    }
    function logout () {
      userMgr!.signoutRedirect();
    }
    return {
      user,
      login,
      logout,
      resourceServerExamples
    }
  }
})
</script>
