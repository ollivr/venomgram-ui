<template>
  <div>

    <div v-if="modal === true">
      <div style="display: flex; position: fixed; top: 11vh; right: 10px">
        <i @click="modal = false; imageSource = ''" class="far fa-times-circle fa-2x" style="color: navy; margin-top: 5px;"></i>
      </div>
      <img v-bind:src="imageSource" alt="user img" style="margin-top: 10vh; max-width: 100vw; max-height: 80vh;">
    </div>

    <div v-if="modal === false">
    <div class="body">
      <div class="container">

        <div v-if="noPosts">
          <p style="padding: 5px;">
            There doesn't seem to be anything here.
            Head to the <router-link to="/explore">Explore page</router-link> to find some people to follow!
          </p>
        </div>

        <div v-for="post in posts" :key="post.id" class="card">
          <div>
            <div style="display: flex; background-color: navy; justify-content: space-between; padding: 10px;">
              <div style="display: flex; justify-content: flex-start; align-items: center; width:33%;">
                <i v-if="post.likedBy.includes(username)" @click="interact('post/unlike', post._id, null, 'PATCH')" class="fas fa-heart"></i>
                <i v-else @click="interact('post/like', post._id, null, 'PATCH')" class="far fa-heart"></i>
                <p style="color: white; display: inline; margin-left: 10px;">{{post.likedBy.length}}</p>
              </div>
              <div style="display: flex; justify-content: center; align-items: center; width:33%;">
                <img v-bind:src="cloudinaryURL + post.username + '.jpg'" alt="user img" width=25 height=25 style="border-radius: 50%;" />
              </div>
              <div style="display: flex; justify-content: flex-end; align-items: center; width:33%;">
                <i v-if="selectedPost !== post._id" @click="selectedPost = post._id" class="fas fa-ellipsis-v"></i>
                <i v-if="selectedPost === post._id" @click="selectedPost = ''" class="fas fa-ellipsis-v"></i>
              </div>
            </div>

            <div v-if="selectedPost === post._id" style="background-color: navy; color: white; display: flex; justify-content: space-around;">
              <p @click="interact('user/unfollow', null, post.username, 'PATCH')" style="cursor: pointer">Unfollow</p>
              <p @click="modal = true; imageSource = post.path" style="cursor: pointer;">Expand</p>
              <p v-if="post.reportedBy.includes(id)" @click="interact('post/report', post._id, null, 'PATCH')" style="cursor: pointer; color: red;">Report</p>
              <p v-if="!post.reportedBy.includes(id)" @click="interact('post/report', post._id, null, 'PATCH')" style="cursor: pointer;">Report</p>
              <p @click="interact('user/block', null, post.username, 'PATCH')" style="cursor: pointer;">Block</p>
            </div>

            <img @dblclick="interact('post/like', post._id, null, 'PATCH')" v-bind:src="post.path" alt="user post" class="picture" />
          </div>
          <div>
            <p style="width:100%; padding: 5px;"><b style="cursor: pointer;" v-on:click="viewUser(post.username)">{{post.username}} </b>{{post.caption}}</p>
          </div>
        </div>

        <button style="margin-bottom:15px;" id="loadMore" v-if="posts.length >= 5 && morePosts === true" @click="loadMore()">
          Load More
        </button>

      </div>
      </div>
    </div>

  </div>
</template>

<script>

export default {
  name: 'Feed',
  data() {
    return {
      posts: [],
      from: 0,
      to: 5,
      morePosts: true,
      modal: false,
      imageSource: '',
      noPosts: false,
      viewProfile: false,
      profileSource: '',
      cloudinaryURL: process.env.VUE_APP_CLOUDINARY,
      selectedPost: '',
    };
  },
  mounted() {
    fetch(`${process.env.VUE_APP_SERVER}/post/feed/${this.username}/${this.from}/${this.to}`)
      .then(res => res.json())
      .then((data) => {
        if (data === 'Authentication Failed.') {
          localStorage.clear();
          this.$store.commit('setUser');
        } else if (data.length === 0) {
          this.noPosts = true;
        }
        this.posts = data;
      })
      .catch(err => console.log(err));
  },
  methods: {
    loadMore() {
      this.from += 5;
      this.to += 5;
      fetch(`${process.env.VUE_APP_SERVER}/post/feed/${this.username}/${this.from}/${this.to}`)
        .then(res => res.json())
        .then((data) => {
          if (data === 'Authentication Failed.') {
            localStorage.clear();
            this.$store.commit('setUser');
          } else if (data.length === 0) {
            this.morePosts = false;
          }
          this.posts.push(...data);
        })
        .catch(err => console.log(err));
    },
    interact(action, postId, username, method) {
      fetch(`${process.env.VUE_APP_SERVER}/${action}`, {
        method,
        headers: { Authorization: `Bearer ${this.token}`, 'Content-Type': 'application/json' },
        body: JSON.stringify({ id: postId, username }),
      })
        .then(res => res.json())
        .then((data) => {
          if (data === 'Authentication Failed.') {
            localStorage.clear();
            this.$store.commit('setUser');
          }
          fetch(`${process.env.VUE_APP_SERVER}/post/feed/${this.username}/0/${this.to}`)
            .then(res => res.json())
            .then((data) => {
              if (data === 'Authentication Failed.') {
                localStorage.clear();
                this.$store.commit('setUser');
              }
              this.posts = data;
            });
        })
        .catch(err => console.log(err));
    },
    viewUser(targetUser) {
      localStorage.setItem('targetUser', targetUser);
      this.$store.commit('targetUser');
      this.$router.push('user');
    },
  },
  computed: {
    username() {
      return this.$store.state.username;
    },
    id() {
      return this.$store.state.id;
    },
    token() {
      return this.$store.state.token;
    },
  },
};

</script>

<style scoped>

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding-bottom: 25px;
  padding-top: 25px;
}

button {
  outline: none;
  border: none;
  background-color: navy;
  color: white;
  font-family: 'Montserrat', Verdana, sans-serif;
  padding: 10px 20px;
  border-radius: 3px;
  transition: 0.1s;
  cursor: pointer;
}

button:hover {
  background-color: #42b883;
}

i {
  color: white;
  padding: 0px 5px;
  cursor: pointer;
}

</style>
