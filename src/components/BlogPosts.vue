<template>
  <!-- Check blog posts exist -->

  <div v-if="posts.length !== 0" class="blog-main">
    <div class="nav-links">
      <button v-on:click="getPosts()">Tous</button>
      <button v-on:click="getPosts('dev')">Dev</button>
      <button v-on:click="getPosts('politique')">Politique</button>
    </div>
    <!-- Template for blog posts -->
    <div v-for="post in posts" :key="post.id" v-bind:post="post" class="blog-post">
      <router-link :to="linkResolver(post)">
        <h2>{{ $prismic.richTextAsPlain(post.data.title) }}</h2>
        <p class="blog-post-meta"><span class="created-at">{{ Intl.DateTimeFormat('en-US', dateOptions).format(new Date(post.data.date)) }}</span></p>
        <div>
          <p>{{getFirstParagraph(post)}}</p>
        </div>
      </router-link>
    </div>
  </div>
  <!-- If no blog posts return message -->
  <div v-else class="blog-main">
    <button v-on:click="getPosts()">Tous</button>
    <button v-on:click="getPosts('dev')">Dev</button>
    <button v-on:click="getPosts('politique')">Politique</button>
    <p>No Posts published at this time.</p>
  </div>
</template>

<script>
export default {
  name: 'blog-posts',
  data () {
    return {
      posts: [],
      dateOptions: { year: 'numeric', month: 'short', day: '2-digit' },
      linkResolver: this.$prismic.linkResolver
    }
  },
  methods: {
    getPosts (tags) {
      if (tags == undefined){
        this.$prismic.client.query(
          this.$prismic.Predicates.at('document.type', "post"),
          { orderings : '[my.post.date desc]' }
        ).then((response) => {
          this.posts = response.results;
        })
      }
      else {
        this.$prismic.client.query(
          this.$prismic.Predicates.at('document.tags', [tags]),
          { orderings : '[my.post.date desc]'}
        ).then((response) => {
          this.posts = response.results;
        })
      }
      //Query to get blog posts

    },
    //Function to get the first paragraph of text in a blog post and limit the displayed text at 300 characters
    getFirstParagraph (post) {
      const textLimit = 300;
      const slices = post.data.body;
      let firstParagraph = '';
      let haveFirstParagraph = false;

      slices.map(function(slice) {
        if (!haveFirstParagraph) {
          if (slice.slice_type == "text") {
            slice.primary.text.forEach(function(block){
              if (block.type == "paragraph") {
                if (!haveFirstParagraph) {
                  firstParagraph += block.text;
                  haveFirstParagraph = true;
                }
              }
            });
          }
        }
      });

      const limitedText = firstParagraph.substr(0, textLimit)

      if (firstParagraph.length > textLimit) {
        return limitedText.substr(0, limitedText.lastIndexOf(' ')) + '...';
      }
      else {
        return firstParagraph;
      }
    },
  },
  created () {
    this.getPosts()
  }
}
</script>

<style scoped>
.nav-links {
  display: flex;
  justify-content: space-around;
  margin-bottom: 30px;
}
.blog-main {
  max-width: 700px;
  margin: auto;
}
.blog-post {
  margin-bottom: 3rem;
}
.blog-post h2 {
  margin: 0;
}

.blog-post-meta {
  color: #9A9A9A;
  font-family: 'Lato', sans-serif;
  margin-bottom: 8px;
  font-size: 16px;
}
/* Media Queries */
@media (max-width: 767px) {
  .blog-main {
    padding: 0 20px;
    font-size: 18px;
  }
}
</style>
