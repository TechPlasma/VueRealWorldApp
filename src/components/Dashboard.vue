<template>
	<div id="dashboard">
		<section>
			<div class="col1">
				<div class="profile">
					<h5>{{ userProfile.name }}</h5>
					<p>{{ userProfile.title }}</p>
					<div class="create-post">
						<p>create a post</p>
						<form @submit.prevent>
							<textarea v-model.trim="post.content"></textarea>
							<button @click="createPost" :disabled="post.content == ''" class="button">post</button>
						</form>
					</div>
				</div>
			</div>
			<div class="col2">
				<transition name="fade">
					<div v-if="hiddenPosts.length" @click="showNewPosts" class="hiddenPosts">
						<p>
							Click to show <span class="new-posts">{{hiddenPosts.length}}</span>
							new <span v-if="hiddenPosts.length > 1">posts</span><span v-else>post</span>
						</p>
					</div>
				</transition>
				<div v-if="posts.length">
					<div v-for="post in posts" class="post">
						<h5>{{post.userName}}</h5>
						<span>{{post.createdOn | formatDate}}</span>
						<p>{{post.content | trimLength}}</p>
						<ul>
							<li><a>comments {{posts.comments}}</a></li>
							<li><a>likes {{posts.likes}}</a></li>
						</ul>
					</div>
				</div>
				<div v-else>
					<p class="no-results">There are currently no posts</p>
				</div>
			</div>
		</section>
	</div>
</template>

<script>
	import { mapState } from 'vuex'
	const fb = require('../firebaseConfig.js')
	import moment from 'moment'

	export default {
		data() {
			return {
				post: {
					content: ''
				}
			}
		},
		computed: {
			...mapState(['userProfile','currentUser','posts','hiddenPosts'])
		},
		methods: {
			createPost() {
				fb.postsCollection.add({
					createdOn: new Date(),
					content: this.post.content,
					userId: this.currentUser.uid,
					userName: this.userProfile.name,
					comments: 0,
					likes: 0
				}).then(ref => {
					this.post.content = ''
				}).catch(err => {
					console.log(err);
				})
			},
			showNewPosts() {
				let updatedPostsAray = this.hiddenPosts.concat(this.posts)
				// clear hiddenPosts array and update posts array
				this.$store.commit('setHiddenPosts', null)
				this.$store.commit('setPosts', updatedPostsAray)
			}
		},
		filters: {
			formatDate(val) {
				if(!val) { return '-' }
				let date = val.toDate()
				return moment(date).fromNow()
			},
			trimLength(val) {
				if(val.length < 200){
					return val
				}
				return `${val.substring(0,200)}...`
			}
		}
	}
</script>