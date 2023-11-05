<template>
  <li>
    <span class="wish" v-on:click="edit();">
      <span
        v-bind:class="{
          active: activeUser === user.name || wish.wisher === activeUser
        }"
      >
        {{ wish.wish }}
        <p v-if="wish.wisher !== wish.name">tips från {{ wish.wisher }}</p>
      </span>
      <button
        class="round edit"
        v-if="user.name === activeUser || wish.wisher === activeUser"
      >
        <img class="editIcon" src="../assets/edit.svg" />
      </button>
    </span>

    <button
      class="round"
      v-if="user.name !== activeUser"
      v-on:click="buy(wish);"
      v-bind:class="{
        unavailable: wish.bought,
        available: !wish.bought
      }"
    >
      <span v-if="wish.bought"> {{ wish.buyer.charAt(0) }} </span>
    </button>
  </li>
</template>

<script>
export default {
  props: ["user", "wish"],
  data() {
    return {
      activeUser: this.$cookie.get("name")
    };
  },
  methods: {
    edit() {
      if (this.wish.wisher === this.activeUser) {
        this.$emit("edit");
      }
    },
    buy() {
      this.$emit("buy");
    },
    getUserWishes(name) {
      return this.allWishes
        .filter(wish => {
          const isWishForUser = wish.name === name;
          const isWishByUser = wish.wisher === name;
          const isWishByActiveUser = wish.wisher === this.activeUser;

          return isWishForUser && (isWishByUser || !isWishByActiveUser);
        })
        .sort((a, b) => a.id - b.id);
    }
  }
};
</script>

<style scoped>
.wish {
  display: flex;
}

li {
  list-style: none;
  text-align: left;
  padding: 0.5em;
  clear: both;
  overflow: hidden;
  font-size: 18px;
  display: flex;
}

span {
  flex: 1;
}

p {
  display: inline;
  font-size: 12px;
  opacity: 0.5;
}

.active {
  border-bottom: dotted rgba(0, 0, 0, 0.2) 2px;
}

#ludwig .active {
  border-bottom: dotted rgba(255, 255, 255, 0.2) 2px;
}

.round {
  text-transform: uppercase;
  border: solid 1px black;
  border-radius: 100%;
  font-weight: bold;
  font-size: 18px;
  height: 2em;
  width: 2em;
  outline: none;
}

.edit {
  background: transparent;
  border: none;
  cursor: pointer;
}

#ludwig .edit {
  filter: invert(0.8);
}

.editIcon {
  height: 100%;
  width: 100%;
}

.available {
  background-color: #01ff70;
}

.unavailable {
  background-color: orangered;
}
</style>
