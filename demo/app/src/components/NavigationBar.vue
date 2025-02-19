<template>
  <div class="ui container">
    <router-link
      to="/"
      class="header item"
    >
      <img
        class="ui mini image"
        src="../assets/logo.png"
      >
      &nbsp;
      Auth-Vue Demo
    </router-link>
    <a
      class="item"
      v-if="!user || !user.profile"
      v-on:click="login()"
    >
    Login
    </a>
    <router-link
      to="/messages"
      class="item"
      id="messages-button"
      v-if="user && user.profile"
    >
      <i
        aria-hidden="true"
        class="mail outline icon">
      </i>
      Messages
    </router-link>
    <router-link
      to="/profile"
      class="item"
      id="profile-button"
      v-if="user && user.profile"
    >
    Profile
    </router-link>
    <a
      id="logout-button"
      class="item"
      v-if="user && user.profile"
      v-on:click="logout()"
    >
    Logout
    </a>
  </div>
</template>

<script lang="ts">
import { User, UserManager } from 'oidc-client'
import { defineComponent, inject, onMounted, ref } from 'vue'

export default defineComponent({
  name: 'app',
  setup () {
    const userMgr = inject<UserManager>('userMgr');
    const user = ref<User>({} as User);

    const fetchUser = () => {
      if (userMgr) {
        userMgr.getUser().then((_user) => {
          if (_user) {
            user.value = _user;
            console.log('fetchUser', user.value)
          }
        });
      }
    }

    // Force page refresh after login/logout
    if (userMgr) {
      userMgr.events.addUserLoaded((_user) => {
        console.log('userMgr.events.addUserLoaded', _user);
        fetchUser();
      });
      userMgr.events.addUserUnloaded(() => {
        console.log('userMgr.events.addUserUnloaded');
        user.value = {} as User;
      });
    }

    onMounted(() => {
      fetchUser();
    });

    const login = () => {
      userMgr!.signinRedirect( { state: window.location.pathname }).then((_user) => {
        console.log('userMgr.signinRedirect', _user);
        fetchUser();
      })
    }

    const logout = () => {
      userMgr!.signoutRedirect( { state: '/' }).then(() => {
        user.value = {} as User;
      })
    }

    return {
      user,
      login,
      logout
    }
  }
})
</script>
