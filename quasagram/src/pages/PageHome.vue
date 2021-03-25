<template>
  <q-page class="constrain q-pa-md">
    <div class="row q-col-gutter-lg">
      <div class="col-12 col-sm-8">
        <!-- CARD PRINCIPAL -->
        <template v-if="loadingPosts === true">
          <q-card
            v-for="post in posts"
            :key="post.id"
            class="my-card q-mb-md"
            flat
            bordered
          >
            <q-item>
              <q-item-section avatar>
                <q-avatar>
                  <img src="/images/ricardo_instagram.jpg" />
                </q-avatar>
              </q-item-section>

              <q-item-section>
                <q-item-label class="text-bold"
                  >ricardo_wojciechowski</q-item-label
                >
                <q-item-label caption>
                  {{ post.location }}
                </q-item-label>
              </q-item-section>
            </q-item>

            <q-separator />

            <q-card class="my-card">
              <q-img :src="post.photo"> </q-img>
            </q-card>

            <q-card-section>
              <div>{{ post.caption }}</div>
              <div class="text-caption text-grey">
                {{ post.date | niceDate }}
              </div>
            </q-card-section>
          </q-card>
        </template>
        <!-- CARD LOADING -->
        <template v-else>
          <q-card flat bordered>
            <q-item>
              <q-item-section avatar>
                <q-skeleton type="QAvatar" animation="fade" size="40px" />
              </q-item-section>

              <q-item-section>
                <q-item-label>
                  <q-skeleton type="text" animation="fade" />
                </q-item-label>
                <q-item-label caption>
                  <q-skeleton type="text" animation="fade" />
                </q-item-label>
              </q-item-section>
            </q-item>
            <q-skeleton height="200px" square animation="fade" />
            <q-card-section>
              <q-skeleton type="text" class="text-subtitle2" animation="fade" />
              <q-skeleton
                type="text"
                width="50%"
                class="text-subtitle2"
                animation="fade"
              />
            </q-card-section>
          </q-card>
        </template>
      </div>
      <!-- COMPONENTE USUÁRIO -->
      <div class="col-4 large-screen-only">
        <q-item class="fixed">
          <q-item-section avatar>
            <q-avatar size="38px">
              <img src="/images/ricardo_instagram.jpg" />
            </q-avatar>
          </q-item-section>

          <q-item-section>
            <q-item-label class="text-bold">ricardo_wojciechowski</q-item-label>
            <q-item-label caption>
              Ricardo Wojciechowski
            </q-item-label>
          </q-item-section>
        </q-item>
      </div>
    </div>
  </q-page>
</template>

<script>
import { date } from "quasar";

export default {
  name: "PageHome",
  data() {
    return {
      posts: [],
      loadingPosts: false
    };
  },
  methods: {
    getPosts() {
      setTimeout(() => {
        this.$axios
          .get("http://localhost:8080/posts")
          .then(response => {
            this.posts = response.data;
            this.loadingPosts = true;
          })
          .catch(error => {
            console.log("error: ", error);
          });
      }, 3000);
    }
  },
  filters: {
    niceDate(value) {
      return date.formatDate(value, "HH:mmaa DD/MMMM/YY ");
    }
  },
  mounted() {
    this.getPosts();
  }
};
</script>

<style lang="scss">
.q-img {
  min-height: 200px;
}
</style>
