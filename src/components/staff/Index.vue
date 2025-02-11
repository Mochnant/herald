<template>
  <div id="page">
    <h2>Staff Control Panel</h2>

    <div class="mt-8">
      <p>Search</p>
      <div class='form-group search'>
        <input type='text' v-model="search_query" @keyup.enter="onSearch"/>
      </div>
    </div>

    <div class="mt-4" v-if="user_results.length">
      <h3>USERS</h3>
      <div v-for="user in user_results" :key="user.id">
        [ {{ user.id }} ]
        <router-link :to="user_details(user)">{{ user.email }}</router-link>
        <span v-if="user.name">- {{ user.name }}</span>
      </div>
    </div>

    <div class="mt-4" v-if="player_results.length">
      <h3>PLAYERS</h3>
      <div v-for="player in player_results" :key="player.id">
        [ {{ player.id }} ]
        <router-link :to="player_details(player)">{{ player.name }}</router-link>
        ({{ player.user_id }})
        -
        <router-link :to="world_details(player)">{{ player.world_name }}</router-link>
      </div>
    </div>



  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import ElementList from "@/components/elementlist/ElementList.vue";
import axios from "axios";
import { BUILDER_WORLD_PLAYER_DETAIL, STAFF_USER_INFO, BUILDER_WORLD_INDEX } from "@/router";

@Component({
  components: {
    ElementList
  }
})
export default class StaffPage extends Vue {
  search_query: string = "";
  user_results: any[] = [];
  player_results: any[] = [];

  timeout: number | null = null;

  user_details(user) {
    return {
      name: STAFF_USER_INFO,
      params: { user_id: user.id },
    };
  }

  player_details(player) {
    return {
      name: BUILDER_WORLD_PLAYER_DETAIL,
      params: { player_id: player.id, world_id: player.world_id },
    };
  }

  world_details(player) {
    return {
      name: BUILDER_WORLD_INDEX,
      params: { world_id: player.world_id },
    };
  }

  @Watch("search_query")
  onSearchQueryChange() {
    if (this.timeout) {
      clearTimeout(this.timeout);
    }
    this.timeout = setTimeout(() => {
      this.onSearch();
    }, 500);
  }

  async onSearch() {
    const query = this.search_query.trim();
    const params = { q: query }

    const resp = await axios.get("staff/search", {
      params: params
    });

    this.user_results = resp.data.users;
    this.player_results = resp.data.players;
  }

}
</script>

<style scoped lang="scss">
@import "@/styles/colors.scss";
@import "@/styles/fonts.scss";
@import "@/styles/layout.scss";

.search { max-width: 320px }
</style>