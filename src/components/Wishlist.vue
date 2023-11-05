<template>
  <main id="main">
    <div v-for="user in users" :key="user.name" :id="user.name" class="user">
      <img v-if="true" class="photo" :src="getImage(user.name)" />
      <h2>{{ user.name }}</h2>
      <ul v-if="activeUser">
        <Wish
          v-for="wish in getUserWishes(user.name)"
          v-bind:key="wish.id"
          :wish="wish"
          :user="user"
          @edit="() => editWish(wish)"
          @buy="() => buyWish(wish)"
        />
        <li v-if="getUserWishes(user.name).length === 0">
          <span class="no-wishes">Inga önskningar ännu!</span>
        </li>
      </ul>
    </div>
  </main>
</template>

<script>
import { sortBy } from "lodash";
import EditDialog from "./EditDialog.vue";
import Wish from "./Wish.vue";
const utils = require("../utils");

export default {
  name: "Wishlist",
  data() {
    return {
      activeUser: this.$cookie.get("name")
    };
  },
  components: {
    EditDialog,
    Wish
  },
  computed: {
    users: function() {
      return this.$store.state.users;
    },
    allWishes: function() {
      return this.$store.state.allWishes;
    },
    isOwner: function(wish) {
      return wish.wisher === this.activeUser;
    }
  },
  methods: {
    getImage(name) {
      return utils.getImage(name);
    },
    getUserWishes: function(name) {
      return this.allWishes
        .filter(
          wish =>
            wish.name === name &&
            (wish.wisher === wish.name ||
              (wish.wisher !== wish.name && name !== this.activeUser))
        )
        .sort((a, b) => a.id - b.id);
    },
    buyWish: async function(wish) {
      if (!wish.bought || wish.buyer === this.activeUser) {
        const res = await fetch("/api/wishes/" + wish.id, {
          method: "POST",
          credentials: "include",
          body: JSON.stringify({
            bought: !wish.bought,
            buyer: !wish.bought ? this.activeUser : ""
          }),
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json"
          }
        });
        const wishes = await res.json();
        this.$store.state.allWishes = wishes;
      }
    },
    editWish: function(wish) {
      this.$modal.show(
        EditDialog,
        {
          wish: wish
        },
        {
          width: "100%",
          draggable: false,
          clickToClose: true,
          height: "auto"
        }
      );
    }
  },
  created: async function() {
    const res_wishes = await this.$http.get("/api/wishes", {
      credentials: "include"
    });
    this.$store.state.allWishes = res_wishes.data;

    const res_users = await this.$http.get("/api/users", {
      credentials: "include"
    });
    this.$store.state.users = sortBy(res_users.data, "id");
  }
};
</script>

<style scoped>
main {
  margin-top: 3em;
}

h2 {
  text-transform: capitalize;
  font-weight: bold;
  margin: 0;
  padding: 0;
}

ul {
  margin: 0;
  padding: 0;
}

.no-wishes {
  text-align: center;
}

.user {
  position: relative;
  padding-top: 4em;
  padding-bottom: 2em;
}

#ludwig {
  background-color: #001f3f;
  color: hsla(210, 100%, 75%, 1);
}
#simon {
  background-color: #7fdbff;
}
#annie {
  background-color: #39cccc;
}
#mamma {
  background-color: #2ecc40;
}
#pappa {
  background-color: #ffdc00;
}

.photo {
  width: 5em;
  border-radius: 50%;
  position: absolute;
  top: -1.5em;
  left: calc(50% - 2.5em);
  filter: saturate(0.5);
  transition: all 1s;
  transform: scale(1);
}

.animate {
  transform: scale(0);
}

@media (min-width: 900px) {
  main {
    display: flex;
  }

  .user {
    width: 20%;
  }
}
</style>
