<template>
  <form v-if="activeUser">
    <div class="header">{{ welcomeText }}</div>
    <div class="form-content">
      <input type="text" :placeholder="placeholder" v-model="newWishForMe" />
      <button @click.prevent="addWishForMe">Önska</button>
    </div>
    <div class="header">Eller önska åt någon annan!</div>
    <div class="form-content">
      <select v-model="selectedUser">
        <option value="" disabled selected>Välj person</option>
        <option
          v-for="user in $store.state.users.filter(
            user => user.name !== this.activeUser
          )"
        >
          {{ user.name }}
        </option>
      </select>
      <input type="text" :placeholder="placeholder" v-model="newWishForOther" />
      <button @click.prevent="addWishForOther">Önska</button>
    </div>
  </form>
</template>

<script>
import { capitalize } from "lodash";
export default {
  data() {
    return {
      activeUser: this.$cookie.get("name"),
      newWishForMe: "",
      newWishForOther: "",
      selectedUser: ""
    };
  },
  computed: {
    placeholder() {
      return "Skriv din önskan här...";
    },
    welcomeText() {
      return " Hej "
        .concat(capitalize(this.activeUser))
        .concat(", vad önskar du dig?");
    }
  },
  methods: {
    addWishForMe() {
      const body = {
        wish: this.newWishForMe,
        name: this.activeUser,
        wisher: this.activeUser
      };
      this.addWish(body);
    },
    addWishForOther() {
      const body = {
        wish: this.newWishForOther,
        name: this.selectedUser,
        wisher: this.activeUser
      };
      this.addWish(body);
    },
    async addWish(body) {
      if (Object.values(body).every(field => field.trim() !== "")) {
        const res = await fetch("/api/wishes/", {
          method: "POST",
          body: JSON.stringify(body),
          credentials: "include",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json"
          }
        });
        const wishes = await res.json();
        this.$store.state.allWishes = wishes;
        this.newWishForMe = this.newWishForOther = "";
      }
    }
  }
};
</script>

<style scoped>
.form-content {
  display: flex;
  justify-content: center;
  width: 80%;
  margin: 0 auto;
}

.header {
  flex: 1;
  color: #515155;
  font-size: 20px;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  margin: 0.5em;
}

input[type="text"] {
  flex: 1;
  box-sizing: border-box;
  height: 2.5em;
  padding: 0 5px;
  border: solid #767676 1px;
}

button {
  margin-left: -10px;
  height: 2.5em;
  width: 5em;
  background: #515155;
  color: white;
  border: 0;
  text-transform: uppercase;
}

::placeholder {
  color: #999999;
}

select {
  height: 2.5em;
  padding: 0 5px;
  width: 8em;
  border-right: 0;
}

@media (min-width: 900px) {
  input[type="text"] {
    max-width: 750px;
  }
}
</style>
