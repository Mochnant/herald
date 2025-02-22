<template>
  <div id="world-config" class="builder-config">
    <h2>{{ world.name.toUpperCase() }} CONFIG</h2>

    <div class="general-settings mt-6">
      <!-- <h3>GENERAL SETTINGS</h3> -->

      <div class="color-text-60">
        <span v-if="world.is_public">Public</span><span v-else>Private</span> World
      </div>

      <div class="color-text-60 mb-6">Publication Status: {{ review_status }} <Help :help="review_help" v-if="review_help"/></div>
      <div class="review-details" v-if="review_status == 'Reviewed'">
        <div class="reviewer color-text-60 mb-2">Comments by {{ review.reviewer }}:</div>
        <div class="review-text mb-4">
          <div class="review-line min-line-height"
              v-for="(line, index) in review.text.split('\n')"
              :key="index">{{ line }}</div>
          </div>
      </div>

      <div v-if="world.description" class="world-description">
        <div class="desc-line" v-for="(line, index) of descLines" :key="index">{{ line }}</div>
      </div>

      <div class="settings-actions mt-4">
        <button class="btn-small mr-4" @click="editGeneral">EDIT</button>
        <button class="btn-small" @click="deleteWorld">DELETE</button>
        <button class="btn-small ml-4" @click="submitForReview" v-if="displaySubmitReview">SUBMIT FOR REVIEW</button>
      </div>
    </div>

    <div class="divider"></div>

    <div class="config-panels">
      <div class="advanced-config">
        <h3>ADVANCED CONFIG</h3>
        <template v-if="config">
          <ul class="list">
            <li>New characters will enter the game with {{ config.starting_gold }} gold.</li>

            <li>
              New characters will enter the game in
              <router-link :to="room_link(config.starting_room.id)">{{ config.starting_room.name }}</router-link>.
            </li>

            <li>
              On death, player will be taken to
              <router-link :to="room_link(config.death_room.id)">{{ config.death_room.name }}</router-link>.
            </li>
            <li>
              Game
              <template v-if="!config.is_narrative">
                allows combat, pvp mode is
                <span v-if="config.pvp_mode === 'free_for_all'">Free for All.</span>
                <span v-else-if="config.pvp_mode === 'zone'">PvP Zones.</span>
                <span v-else>Disabled.</span>
              </template>
              <template v-else>is a narrative world, does not allow combat.</template>
            </li>

            <li v-if="config.small_background || config.large_background">
              <div v-if="config.small_background">
                General Lobby art:
                <a
                  :href="config.small_background"
                  v-if="config.small_background.startsWith('http')"
                >link</a>
                <span v-else>{{ config.small_background }}</span>
              </div>
              <div v-if="config.large_background">
                World Lobby art:
                <a
                  :href="config.large_background"
                  v-if="config.large_background.startsWith('http')"
                >link</a>
                <span v-else>{{ config.large_background }}</span>
              </div>
            </li>
          </ul>

          <button class="btn-thin" @click="editAdvancedConfig">EDIT</button>
        </template>
      </div>

      <div class="world-status">
        <h3>World Status</h3>

        <div>View connected players, start/stop multiplayer worlds.</div>

        <router-link :to="world_status_link">manage</router-link>
      </div>

      <div class="random-profiles">
        <h3>RANDOM ITEM PROFILES</h3>

        <div>
          <p>Random Item Profiles offer a way to define a random load. Use cases include:</p>
          <ul class="list">
            <!-- <li>Equipping a mob with random gear</li> -->
            <li>Giving a random item reward on completing a quest</li>
            <li>Merchants with random sales inventory</li>
          </ul>
        </div>

        <router-link
          :to="{name: BUILDER_WORLD_RANDOM_PROFILES, params: {world_id: $route.params.world_id}}"
        >manage</router-link>
      </div>

      <div class="transformation-templates">
        <h3>Transformations</h3>

        <div>
          <p>Transformations can be applied to the output of a loader rule to modify a loaded template. Use cases include:</p>
          <ul class="list">
            <li>Making a mob roam 100% of the time on tic rather than the default 5%</li>
            <li>Force a mob to roam in a particular direction</li>
            <li>Change the name of a mob when it loads</li>
            <li>Make any other one-off modification to a template for a loaded mob.</li>
          </ul>
        </div>

        <router-link
          :to="{name: BUILDER_WORLD_TRANSFORMATION_TEMPLATE_LIST, params: {world_id: $route.params.world_id}}"
        >manage</router-link>
      </div>

      <div class="world-builders">
        <h3>World Builders</h3>

        <div>Builders are able to access the editor for a given world. They can be given read-only access.</div>

        <router-link
          :to="{name: BUILDER_WORLD_BUILDERS, params: {world_id: $route.params.world_id}}"
        >manage</router-link>
      </div>

      <div class="world-players">
        <h3>World Players</h3>

        <div>View information about players in your world.</div>

        <router-link
          :to="{name: BUILDER_WORLD_PLAYER_LIST, params: {world_id: $route.params.world_id}}"
        >manage</router-link>
      </div>

      <div class="world-factions">
        <h3>Worlds Factions</h3>

        <div>View information about factions in your world.</div>

        <router-link :to="world_factions_link">manage</router-link>
      </div>

      <div class="world-facts">
        <h3>Worlds Facts</h3>

        <div>Facts are data points about your world that can be set by builders, mobs and a fact schedule. Conditions can then look at those facts to determine which loaders, room actions, quests and reactions should be considered active.</div>

        <router-link :to="world_facts_link">manage</router-link>
      </div>

      <div class='world-skills'>
        <h3>Custom Skills</h3>

        <div>Builders can create skills, usable by players and mobs that do not have an archetype. To enable the creation of players that do not have an archetype, check the "Classless Players" checkbox under Advanced Config.</div>

        <router-link :to="world_skills_link">manage</router-link>
      </div>

      <div class="world-starting-eq">
        <h3>Starting EQ</h3>

        <div>Define the items that a player starts with.</div>

        <router-link :to="world_starting_eq_link">manage</router-link>
      </div>
    </div>
  </div>
</template>

<script lang='ts'>
import { Component, Prop, Vue, Watch, Mixins } from "vue-property-decorator";
import ElementList from "@/components/elementlist/ElementList.vue";
import {
  BUILDER_WORLD_RANDOM_PROFILES,
  BUILDER_WORLD_TRANSFORMATION_TEMPLATE_LIST,
  BUILDER_WORLD_BUILDERS,
  BUILDER_WORLD_PLAYER_LIST,
  BUILDER_ROOM_INDEX,
  BUILDER_WORLD_STATUS,
  BUILDER_WORLD_FACTIONS,
  BUILDER_WORLD_FACTS,
  BUILDER_WORLD_SKILLS,
  BUILDER_WORLD_STARTING_EQ,
  LOBBY,
} from "@/router";
import { capfirst } from "@/core/utils.ts";
import Help from "@/components/Help.vue";
import { BUILDER_FORMS, FormElement } from "@/core/forms";
import { UI_MUTATIONS } from "@/constants";
import WorldView from "@/components/builder/world/WorldView.ts";
import ReviewInstructions from "@/components/builder/world/ReviewInstructions.vue";

@Component({
  components: {
    ElementList,
    Help,
  },
})
export default class WorldFrame extends Mixins(WorldView) {
  BUILDER_WORLD_RANDOM_PROFILES = BUILDER_WORLD_RANDOM_PROFILES;
  BUILDER_WORLD_TRANSFORMATION_TEMPLATE_LIST = BUILDER_WORLD_TRANSFORMATION_TEMPLATE_LIST;
  BUILDER_WORLD_BUILDERS = BUILDER_WORLD_BUILDERS;
  BUILDER_WORLD_PLAYER_LIST = BUILDER_WORLD_PLAYER_LIST;

  room_link(id) {
    return {
      name: BUILDER_ROOM_INDEX,
      params: {
        world_id: this.world.id,
        room_id: id,
      },
    };
  }

  get config() {
    return this.$store.state.builder.worlds.config;
  }

  get world() {
    return this.$store.state.builder.world;
  }

  async fetch() {
    this.$store.commit("builder/worlds/config_clear");
    await this.$store.dispatch("builder/worlds/config_fetch", {
      world_id: this.world.id,
    });
  }

  async editGeneral() {
    const modal = {
      title: `Edit World`,
      data: this.world,
      schema: [
        BUILDER_FORMS.NAME,
        BUILDER_FORMS.DESCRIPTION,
        {
          attr: "is_public",
          label: "Is Public",
          widget: "checkbox",
          help: `A public world is visible and searchable to all players. A private world is only visible to players who have been given access to it.`,
        },
      ],
      action: "builder/world_save",
    };
    this.$store.commit(UI_MUTATIONS.MODAL_SET, modal);
  }

  async editAdvancedConfig() {
    const starting_gold: FormElement = {
      attr: "starting_gold",
      label: "Starting Gold",
    };
    const death_room: FormElement = {
      attr: "death_room",
      label: "Death Room",
      widget: "reference",
      references: "room",
      required: true,
    };
    const starting_room: FormElement = {
      attr: "starting_room",
      label: "Starting Room",
      widget: "reference",
      references: "room",
      required: true,
      help: `Which room a new player starts in by default.<br/><br/>
             A starting room can also be defined at the Faction level,
             which will take precendence over this default starting room.`,
    };
    const death_mode: FormElement = {
      attr: "death_mode",
      label: "Death EQ Loss",
      widget: "select",
      options: [
        {
          value: "lose_all",
          label: "Lose All",
        },
        {
          value: "lose_none",
          label: "Lose None",
        },
        {
          value: "lose_gold",
          label: "Lose Gold",
        },
      ],
      help: `Determines what happens to the player's equipment on death.<br/><br/>
             Lose None: player retains all of their equipment<br/>
             Lose All: all of the player's equipment goes to their corpse<br/>
             Lose Gold: player pays 20% of their equipment's value on death`,
    };
    const built_by: FormElement = {
      attr: "built_by",
      label: "Built By",
      help: `What to show in the 'built by' field of the World Lobby. If missing, will default to the author's username.`,
    };
    const death_route: FormElement = {
      attr: "death_route",
      label: "Death Route",
      widget: "select",
      options: [
        {
          value: "top_faction",
          label: "Top Faction",
        },
        {
          value: "near_room",
          label: "Nearest Room",
        },
        {
          value: "far_room",
          label: "Furthest Room",
        },
        {
          value: "nearest_in_zone",
          label: "Nearest in Zone",
        },
      ],
      help: `Where players go on death.<br/><br/>
             Top Faction: the nearest procession room of the faction you have highest standing with.<br/>
             Nearest Room: the procession room nearest where you died.<br/>
             Furthest Room: the procession room furthest from where you died.<br/>
             Nearest in Zone: the procession room closest to you in your current zone.`,
    };
    const auto_equip: FormElement = {
      attr: "auto_equip",
      label: "Auto Equip Items",
      widget: "checkbox",
      help: `If checked, items acquired to the player's inventory will
             automatically equip if the corresponding slot is empty.`,
    };
    const allow_pvp: FormElement = {
      attr: "allow_pvp",
      label: "Allow PvP",
      widget: "checkbox",
      help: `If checked, players can kill other players.`,
    };
    const pvp_mode: FormElement = {
      attr: "pvp_mode",
      label: "PvP Mode",
      widget: "select",
      options: [
        {
          value: "free_for_all",
          label: "Free for All",
        },
        {
          value: "disabled",
          label: "Disabled",
        },
        {
          value: "zone",
          label: "PvP Zones",
        },
      ],
      help: `In multiplayer worlds, to what extent PvP is allowed.<br/><br/>
            Free for All - anyone can attack anyone else, unless in a peace room.<br/>
            Disabled - no player can attack another player, ever.<br/>
            PvP Zones - default is no PvP but certain zones can enable it.<br/>
      `,
    };
    const can_select_faction: FormElement = {
      attr: "can_select_faction",
      label: "Can Select Core Faction",
      widget: "checkbox",
      help: `If unchecked, all players will always start with the default core faction.`,
    };
    const allow_combat: FormElement = {
      attr: "is_narrative",
      label: "Narrative World",
      widget: "checkbox",
      help: `A narrative world disables combat, and will not show combat-related UI elements.`,
    };
    const players_can_set_title: FormElement = {
      attr: "players_can_set_title",
      label: "Players Can Set Title",
      widget: "checkbox",
      help: `Whether players are allowed to change their own title.`,
    };
    const small_background: FormElement = {
      attr: "small_background",
      label: "740 x 332 Card URL",
      help: `Image displayed in the general lobby`,
    };
    const large_background: FormElement = {
      attr: "large_background",
      label: "2300 x 598 Banner URL",
      help: `Image displayed in the world lobby`,
    };
    const is_classless: FormElement = {
      attr: "is_classless",
      label: "Classless Players",
      widget: "checkbox",
      help: `If this option is checked, no player will have a starting archetype, or its associated skills. Builders will have to create all the skills that players can learn.`
    };
    const non_ascii_names: FormElement = {
      attr: "non_ascii_names",
      label: "Allow Non-ASCII Names",
      widget: "checkbox",
      help: `If this option is checked, players will be able to use non-ASCII characters in their names.`
    }

    const modal = {
      title: `Edit World Config`,
      data: this.config,
      schema: [
        {
          children: [starting_gold, starting_room],
        },
        {
          children: [death_mode, death_room],
        },
        {
          children: [pvp_mode, death_route],
        },
        {
          children: [allow_combat, auto_equip],
        },
        {
          children: [can_select_faction, players_can_set_title],
        },
        {
          children: [is_classless, non_ascii_names]
        },
        {
          children: [small_background, large_background],
        },
        built_by,
      ],
      action: "builder/worlds/config_save",
    };
    this.$store.commit(UI_MUTATIONS.MODAL_SET, modal);
  }

  async submitForReview() {
    const modal = {
      title: 'Submit For Review',
      data: { 'description': '' },
      submitLabel: 'SUBMIT',
      schema: [
        {
          attr: 'description',
          label: 'Description',
          widget: 'textarea',
          help: `Describe your world to the reviewer.`
        }
      ],
      action: "builder/worlds/submit_world_for_review",
      slot: ReviewInstructions,
    };
    this.$store.commit(UI_MUTATIONS.MODAL_SET, modal);

    //await this.$store.dispatch("builder/worlds/submit_world_for_review");
  }

  async deleteWorld() {
    const world_id = this.world.id;

    // Crude confirm dialog
    const c = confirm(
      `Are you sure you want to delete this world and everything in it? This action cannot be undone.`
    );
    if (!c) return;

    await this.$store.dispatch("builder/world_delete");
    this.$store.commit(
      UI_MUTATIONS.SET_NOTIFICATION,
      `Deleted World ${world_id}`
    );
    this.$router.push({ name: LOBBY });
  }

  get world_status_link() {
    return {
      name: BUILDER_WORLD_STATUS,
      params: { world_id: this.world.id },
    };
  }

  get world_factions_link() {
    return {
      name: BUILDER_WORLD_FACTIONS,
      params: { world_id: this.world.id },
    };
  }

  get world_facts_link() {
    return {
      name: BUILDER_WORLD_FACTS,
      params: { world_id: this.world.id },
    };
  }

  get world_skills_link() {
    return {
      name: BUILDER_WORLD_SKILLS,
      params: { world_id: this.world.id }
    }
  }

  get world_starting_eq_link() {
    return {
      name: BUILDER_WORLD_STARTING_EQ,
      params: { world_id: this.world.id }
    }
  }

  get descLines() {
    return this.world.description.split("\n");
  }

  get displaySubmitReview() {
    return (this.world.review.status === "unsubmitted" || this.world.review.status == "reviewed");
  }

  get review() {
    return this.world.review;
  }

  get review_status() {
    if (this.world.review.status === 'unsubmitted') {
      return 'Unpublished';
    } else if (this.world.review.status === 'submitted') {
      return 'Under Review';
    } else if (this.world.review.status === 'reviewed') {
      return 'Reviewed';
    } else if (this.world.review.status === 'approved') {
      return 'Published';
    }
    return capfirst(this.world.review.status);
  }

  get review_help() {
    if (this.world.review.status === 'unsubmitted') {
      return `A world that's been approved for publication will be featured in curated sections of the site. To initiate a review, click the SUBMIT FOR REVIEW action.`;
    } else if (this.world.review.status === 'submitted') {
      return `Your review has been submitted. Once a staff member reviews it, it will either be approved or you will receive feedback on what to change.`;
    } else if (this.world.review.status === 'reviewed') {
      return `Your world has been reviewed but is not quite ready for primetime yet. Read the reviewer's notes and re-submit it once you're ready.`;
    }
    return '';
  }
}
</script>

<style lang="scss" scoped>
@import "@/styles/colors.scss";
@import "@/styles/layout.scss";

.world-description {
  div.desc-line:not(:last-child) {
    margin-bottom: 0.8em;
  }
}


.review-details {
  .review-text {
    border: 1px solid $color-background-light-border;
    padding: 15px;
  }
}

.divider {
  margin-top: 50px;
  margin-bottom: 50px;
}
</style>