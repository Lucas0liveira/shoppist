<template>
  <form class="form-widget" @submit.prevent="updateProfile">
    <Avatar v-model:path="avatar_url" @upload="updateProfile" />
    <div>
      <label for="email">Email</label>
      <input id="email" type="text" :value="store.user.email" disabled />
    </div>
    <div>
      <label for="username">Username</label>
      <input id="username" type="text" v-model="username" />
    </div>
    <div>
      <label for="fullname">Fullname</label>
      <input id="fullname" type="fullname" v-model="fullname" />
    </div>

    <div>
      <input
        type="submit"
        class="button block primary"
        :value="loading ? 'Loading ...' : 'Update'"
        :disabled="loading"
      />
    </div>

    <div>
      <button class="button block" @click="signOut" :disabled="loading">
        Sign Out
      </button>
    </div>
  </form>
</template>

<script>
import { supabase } from "../supabase";
import { store } from "../store";
import { onMounted, ref } from "vue";
import Avatar from "../components/Avatar.vue";

export default {
  setup() {
    const loading = ref(true);
    const username = ref("");
    const fullname = ref("");
    const avatar_url = ref("");

    async function getProfile() {
      try {
        loading.value = true;
        const user = supabase.auth.user();
        store.user = user;

        if (user) {
          username.value = user.user_metadata.name;
          fullname.value = user.user_metadata.full_name;
          avatar_url.value = user.user_metadata.avatar_url;
        }
      } catch (error) {
        alert(error.message);
      } finally {
        loading.value = false;
      }
    }

    async function updateProfile() {
      try {
        loading.value = true;
        store.user = supabase.auth.user();

        const updates = {
          id: store.user.id,
          username: username.value,
          fullname: fullname.value,
          avatar_url: avatar_url.value,
          updated_at: new Date(),
        };

        let { error } = await supabase.from("profiles").upsert(updates, {
          returning: "minimal", // Don't return the value after inserting
        });

        if (error) throw error;
      } catch (error) {
        alert(error.message);
      } finally {
        loading.value = false;
      }
    }

    async function signOut() {
      try {
        loading.value = true;
        let { error } = await supabase.auth.signOut();
        if (error) throw error;
      } catch (error) {
        alert(error.message);
      } finally {
        loading.value = false;
      }
    }

    onMounted(() => {
      getProfile();
    });

    return {
      store,
      loading,
      username,
      fullname,
      avatar_url,

      updateProfile,
      signOut,
    };
  },
  components: {
    Avatar,
  },
};
</script>
