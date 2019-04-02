<template>
    <div class="container">
        <div class="row">
            <div class="col-md-10 col-md-offset-1">
                <div class="panel panel-default">

                    <div class="panel-heading">Upload a Video</div>

                    <div class="panel-body">

                        <p class="alert text-center alert-warning">Choose a Category &amp; Paste <a href="https://www.youtube.com/" target="_blank">Youtube</a> video link.</p>

                        <div class="row">
                            <div class="col-md-8">
                                <form method="post" @submit.prevent="uploadVideo()" class="form-horizontal">

                                    <div class="form-group">
                                        <label for="category" class="col-sm-2 control-label">Category</label>
                                        <div class="col-sm-10">
                                            <select v-model="video.category_id" required name="category" class="form-control" id="category">
                                                <option :value="cat.id" v-for="cat in categories" :key="cat.id">{{ cat.name }}</option>
                                            </select>
                                        </div>
                                    </div>
                                     <div class="form-group">
                                        <label for="channel" class="col-sm-2 control-label">Channel</label>
                                        <div class="col-sm-10">
                                            <select v-model="video.channel_id" required name="channell" class="form-control" id="channel">
                                                <option :value="cha.id" v-for="cha in channels" :key="cha.id">{{ cha.name }}</option>
                                            </select>
                                        </div>
                                    </div>

                                    <div class="form-group">
                                        <label for="url" class="col-sm-2 control-label">Video Path</label>
                                        <div class="col-sm-10">
                                            
                                            <input v-model="video.url" @blur="validateYoutubeUrl()" required type="text" class="form-control">
                                             
                                        </div>
                                     </div>
                                     <div class="form-group" v-if="movieId">
                                        <label for="url" class="col-sm-2 control-label">Trailer url</label>
                                        <div class="col-sm-10">
                                            
                                            <input  v-model="imageThumb" @blur="validateTrailerUrl()"  type="text" class="form-control">
                                             
                                        </div>
                                    </div>

                                    <div class="form-group" v-if="relatedId">
                                        <label for="category" class="col-sm-2 control-label">Video Relacionado</label>
                                        <div class="col-sm-10">
                                            <select v-model="video.related_id" required name="related" class="form-control" id="related">
                                                <option :value="video.id" v-for="video in videosId.data" :key="video.id">{{ video.title }}</option>
                                            </select>
                                        </div>
                                    </div>

                                    <div class="form-group">
                                        <label for="title" class="col-sm-2 control-label">Title</label>
                                        <div class="col-sm-10">
                                            <input v-model="video.title" required type="text" class="form-control" id="title" placeholder="Title of Video">
                                        </div>
                                    </div>

                                    <div class="form-group">
                                        <label for="desc" class="col-sm-2 control-label">Description</label>
                                        <div class="col-sm-10">
                                            <textarea v-model="video.description" minlength="10" required class="form-control" id="desc" placeholder="Description for video"></textarea>
                                        </div>
                                    </div>

                                    <div class="form-group">
                                        <div class="col-sm-offset-2 col-sm-10">
                                            <button :disabled="loading" type="submit" class="btn btn-info">
                                                <span v-show="loading" class="glyphicon glyphicon-refresh spin"></span> Upload
                                            </button>
                                            <button type="reset" :disabled="loading" class="btn btn-default">Cancel</button>
                                        </div>
                                    </div>
                                </form>
                            </div>
                            <!-- End Video upload form -->

                            <div class="col-md-4">
                                <img class="img-responsive" :src="videoThumb" alt="">
                            </div>
                            <!-- End Video thumb preview -->
                        </div>
                    </div>

                    <div class="panel-heading">Trending Videos</div>

                    <div class="panel-body">
                        <video-thumb :list="videos.data"></video-thumb>
                    </div>
                    <!-- End Latest videos -->
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    
           
    export default {
        data() {
            return {
                user: this.$root.auth.id,
                video: {},
                videos: {
                    data: []
                },
                categories: [],
                channels:[],
                loading: false,
                videoThumb: '/img/video-thumb-placeholder.png',
                movieId: false,
                relatedId: false,
                imageThumb: '',
                videosId: {
                    data: []
                },
            }
        },

        mounted() {
            this.getVideoAndCategories();      
            this.getVideos();     
            console.log('Upload Component mounted.', this.videos)
        },

        methods: {
            getVideoAndCategories() {
                this.$Progress.start();
                // change the title of page
                window.document.title = 'Upload a Video on Pelis Chulas';

                axios.get('/api/videos?trending=true&categories=true&user=' + this.user).then((res) => {
                    this.$Progress.finish();
                    this.videos = res.data;
                    this.categories = res.data.categories;
                    this.channels = res.data.channels;
                }).catch((err) => {
                    this.$Progress.finish();
                    console.log(err);
                });
                
            },

            getVideos(){
                this.$Progress.start();
                 axios.get('/api/videos').then((res) => {
                this.$Progress.finish();
                this.videosId = res.data;
            }).catch((err) => {
                this.$Progress.finish();
                console.log(err);
            });
            },

            uploadVideo() {
                let vm = this;
                vm.loading = true;

                // add other fields
                vm.video.thumbnail = vm.videoThumb;
                
                console.log('post:', vm.video);

                axios.post('/api/videos', vm.video).then(function (res) {
                    vm.video = {};
                    vm.loading = false;
                    alert('Video has been uploaded, Go to home page to see it.');
                }).catch(function (error) {
                    console.log(error);
                    vm.loading = false;
                });
            },

            validateYoutubeUrl() {
              let videoCode = this.isYoutube(this.video.url);              
              if ( videoCode ) {
                  this.videoThumb = 'http://img.youtube.com/vi/'+ videoCode +'/mqdefault.jpg';
                  this.movieId = false;
                  this.relatedId = true;                  
              } else {                                     
                   this.movieId = true;
              }
            },

            validateTrailerUrl() {
              let videoCode = this.isYoutube(this.imageThumb);              
              if ( videoCode ) {
                  this.videoThumb = 'https://img.youtube.com/vi/'+ videoCode +'/mqdefault.jpg';
              } else {                                     
                    alert('URL is not a valid youtube video url.');
              }
            },

            onFileChange(e) {
                var files = e.target.files || e.dataTransfer.files;
                if (!files.length)
                    return;
                var reader = new FileReader();
                reader.readAsDataURL(files[0]);
                console.log(reader.result, e.target.files);
            },   
   

            isYoutube(url) {
                let pattern = /^(?:https?:\/\/)?(?:www\.)?(?:youtu\.be\/|youtube\.com\/(?:embed\/|v\/|watch\?v=|watch\?.+&v=))((\w|-){11})(?:\S+)?$/;
                let matches = url.match(pattern);
                if(matches){
                    return matches[1];
                }
                return false;
            },
        }
    }
</script>
