<template>
  <!-- removing flex and flex center stops everything to be centered within the home page-->
  <!-- pa is padding all and md is for medium padding -->
  <q-page class="constrain q-pa-md">
    <!-- adding col for width -->
    <!-- gutter to add space between each col -->
    <div class="row q-col-gutter-lg">
      <div class="col-12 col-sm-8">
        <!-- avatar -->
        <!-- v-for will link the array-->
        <template v-if="!loadingPosts && posts.length">
          <q-card
            v-for="post in posts"
            :key="post.id"
            class="card-post q-mb-md"
            flat
            bordered
          >
            <q-item>
              <q-item-section avatar>
                <q-avatar>
                  <img :src="post.imageurl" />
                </q-avatar>
              </q-item-section>
              <!-- title and subheading -->
              <q-item-section>
                <q-item-label class="text-bold text-primary">{{
                  post.username
                }}</q-item-label>
                <q-item-label caption>
                  {{ post.location }}
                </q-item-label>
              </q-item-section>
            </q-item>
            <q-separator />
            <!-- image bellow avatar card -->
            <q-img :src="post.imageurl" />
            <!-- caption of image -->
            <q-card-section>
              <div class="text-h6">{{ post.caption }}</div>
              <div class="text-caption text-primary">
                {{ niceDate(post.date) }}
              </div>
            </q-card-section>
          </q-card>
        </template>
        <template v-else-if="!loadingPosts && !posts.length">
          <h5 class="text-center text-pink-10">No Posts Yet.</h5>
        </template>
        <template v-else>
          <q-card>
            <q-item>
              <q-item-section avatar>
                <q-skeleton type="QAvatar" size="40px" />
              </q-item-section>

              <q-item-section>
                <q-item-label>
                  <q-skeleton type="text" />
                </q-item-label>
                <q-item-label caption>
                  <q-skeleton type="text" />
                </q-item-label>
              </q-item-section>
            </q-item>

            <q-skeleton height="200px" square />

            <q-card-actions align="right" class="q-gutter-md">
              <q-skeleton type="QBtn" />
              <q-skeleton type="QBtn" />
            </q-card-actions>
          </q-card>
        </template>
      </div>
      <div class="col-4 large-screen">
        <q-item class="fixed">
          <q-item-section avatar>
            <q-avatar size="40px">
              <img src="../assets/game.jpeg" />
            </q-avatar>
          </q-item-section>
          <!-- title and subheading -->
          <q-item-section>
            <q-item-label class="text-bold text-primary"
              >Cristiana Simoes</q-item-label
            >
          </q-item-section>
        </q-item>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'

import { date } from 'quasar'

export default defineComponent({
  name: 'PageHome',
  data() {
    //data method that returns an object this object is the captions, locations, date and image
    return {
      posts: [], // posts start as empty array, we load them at mounted. You can also add something like, if loading is false, but !posts.length (posts is empty) then show a message like "No posts found." and maybe a button to try loading again.
      loadingPosts: false
    }
  },
  async mounted() {
    // this happens when the component is ready to go (mounted)! again, we are using "async" before it so we can use await inside of it.
    // we are starting to load, you can set for example: this.loading = true;
    this.loadingPosts = true

    await this.load() // we await our load method
    // here we are finished with the loading, you can set for example: this.loading = false;
    this.loadingPosts = false
  },
  methods: {
    async load() {
      // we're making this async just so we can use await inside. async / await is a cleaner way of using promises (async code)
      const response = await this.$axios
        .get('https://www.reddit.com/r/itookapicture/top.json?t=month')
        .then(res => res.data.data.children) // we make the GET request with this.$axios.get, GET requests is what the browser does when accessing an URL. We have a .then that helps us get to the information that we really need. We could do this outside the .then as well. our const response will get this result.
      this.posts = response.map(post => {
        // .map helps us transform our array. response is our array of (25) posts, but they are in the Reddit format. We want to transform them to our format and grab what we need.
        const { id, author, title, created, url, city } = post.data // each post has a .data property with some interesting stuff we can grab. here we are destructuring it to some variables that interest us.
        return {
          // return a JSON, but OUR json the way we want it!
          id: id,
          username: author, // grab the username
          caption: title, // title
          date: created * 1000, // they store timestamps like this, not really sure why
          location: city, // not using this for now
          imageurl: url // url seems like the proper field from the reddit json
        }
      })
      // after this, this.posts (our variable from data above) will have an array with the JSON we mapped, we just have to use it on our template!
    },
    niceDate(value) {
      return date.formatDate(value, 'MMMM D h:mmA')
    }
  }
})
</script>

<style lang="sass">
.card-post, .q-img
  min-height: 200px
</style>
