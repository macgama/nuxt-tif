<template>
    <div class="container-fluid">
        <!-- The monAvatar -->
        <div id="modalBox">
            <div class="modal-dialog modal-lg">
                <div class="modal-content">
                    <!-- Modal Header -->
                    <div class="modal-header text-center">
                        <span class="modal-title">Ton avatar</span>
                        <nuxt-link to="/home">
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true" class="white-text"><i class="fa fa-arrow-circle-left"></i></span>
                            </button>
                        </nuxt-link>         
                    </div>
                    <!-- Modal body -->
                    <div id="modalBoxContent" class="modal-body">
                        <div class="flex-container-modal-MyTeam">
                            <h1>Envie de changer de tête ?</h1>
                            <!-- <h2> arr: {{ this.arr }} <br /><br />obj: {{ this.obj }}<br /><br />user: {{ this.loadedUser.avatar.name }}</h2> -->
                            <!-- <h2>{{ loadedUser }}</h2> -->
                        </div>
                        <div class="flex-container-modalAvatar" v-if="this.arr.length > 0">
                            <!-- <div style="flex-grow: 1"><img src="/images/avatar.png" class="imgModalAvatar"/></div> -->
                            <div style="flex-grow: 1"><img src="" ref="mergedImage" class="imgModalAvatar" width="300px" /></div>
                        </div>
                        <div class="flex-container-modalAvatar" v-else>
                            <h4 style="color: orangered; margin: 0 auto;">Please select a property below</h4><br /><br />
                        </div>
                        <div class="flex-container-modalMenuAvatar">
                            <div style="flex-grow: 1; cursor: pointer;" :class="{active: this.gender === 'female'}" @click="selectGender('female')"><span class="textModalMenuAvatar">FEMALE</span></div>
                            <div style="flex-grow: 1; cursor: pointer;" :class="{active: this.gender === 'male'}" @click="selectGender('male')"><span class="textModalMenuAvatar">MALE</span></div>
                        </div>
                        <div class="flex-container-modalMenuAvatar">
                            <div style="flex-grow: 1; cursor: pointer;" :class="{active: this.bodyPart === 'background'}" @click="selectBodyPart('background')"><span class="textModalMenuAvatar">BACKGROUND</span></div>
                            <div style="flex-grow: 1; cursor: pointer;" :class="{active: this.bodyPart === 'body'}" @click="selectBodyPart('body')"><span class="textModalMenuAvatar">BODY</span></div>
                            <div style="flex-grow: 1; cursor: pointer;" :class="{active: this.bodyPart === 'skin'}" @click="selectBodyPart('skin')"><span class="textModalMenuAvatar">SKIN</span></div>
                            <div style="flex-grow: 1; cursor: pointer;" :class="{active: this.bodyPart === 'face'}" @click="selectBodyPart('face')"><span class="textModalMenuAvatar">FACE</span></div>
                            <div style="flex-grow: 1; cursor: pointer;" :class="{active: this.bodyPart === 'hair'}" @click="selectBodyPart('hair')"><span class="textModalMenuAvatar">HAIR</span></div>
                        </div>
                        <div class="flex-container-modalAvatarImg">
                            <div v-for="avatar in loadedAvatars" style="cursor: pointer;" @click="addToMerge(avatar.gender, avatar.type, avatar.image, avatar.name)"><img :src="'/images/avatars/' + avatar.gender + '/' + avatar.type + '/' + avatar.image" class="imgModalAvatar" :class="{active: (avatar.name === background ||  avatar.name === body || avatar.name === skin || avatar.name === face || avatar.name === hair) }" /></div>
                        </div>
                    </div>
                    <!-- Modal footer -->
                    <div class="modal-footer">
                        <div class="progress" style="width: 50%; margin: 0 auto;" v-if="arr.length > 0">
                            <div class="progress-bar bg-success" role="progressbar" :style="{width: progress + '%'}" :aria-valuenow="progress" aria-valuemin="0" aria-valuemax="100"></div>
                        </div>
                        <button class="btn btn-success" data-dismiss="modal" :disabled="loading" :loading="loading" @click="saveImage">Allez, valide ! <i v-bind:class="{'fa fa-spinner fa-spin' : loading}"></i></button>
                        <nuxt-link to="/home"><button type="button" class="btn btn-danger" data-dismiss="modal">Annule tout !</button></nuxt-link>
                    </div>
                </div>
            </div><!-- /.modal-dialog -->
        </div><!-- /#modalBox -->
        <!-- End monAvatar -->
    </div><!-- /.container-fluid -->
</template>

<script>
    import firebase from 'firebase'
    import mergeImages from 'merge-images'
    import moment from 'moment'
    import Noty from 'noty'
    export default {
        layout: 'layoutFront',
        created () {
            this.$store.dispatch('posts/loadedPosts')

            const avatarsArray = []
            firebase.database().ref('avatars').once('value', function (snapshot) {
                // console.log(snapshot.val())
                for (const key in snapshot.val()) {
                    avatarsArray.push({...snapshot.val()[key]})
                }
                console.log(avatarsArray)
            })
            this.avatars = avatarsArray

            // const array = []
            if (this.$store.getters['users/loadedUser'].avatar) {
                const array = this.$store.getters['users/loadedUser'].avatar.name.split('_')
                if (array.length >= 7) {
                    this.gender = ''
                    this.gender = array[1]
                    this.background = array[2]
                    this.body = array[3]
                    this.skin = array[4]
                    this.face = array[5]
                    this.hair = array[6]

                    this.obj = [{ "image": '/images/avatars/' + this.gender + '/background/' + this.background + '.png', 'gender': this.gender, 'type': 'background' }, { 'image': '/images/avatars/' + this.gender + '/skin/' + this.skin + '.png', 'gender': this.gender, 'type': 'skin' }, { 'image': '/images/avatars/' + this.gender + '/body/' + this.body + '.png', 'gender': this.gender, 'type': 'body' }, { 'image': '/images/avatars/' + this.gender + '/hair/' + this.hair + '.png', 'gender': this.gender, 'type': 'hair' }, { 'image': '/images/avatars/' + this.gender + '/face/' + this.face + '.png', 'gender': this.gender, 'type': 'face' }]
                    this.mergeImages()
                }
                console.log(array)
            }
        },
        data () {
            return {
                loading: false,
                gender: 'female',
                bodyPart: 'background',
                name: '',
                background: '',
                body: '',
                skin: '',
                face: '',
                hair: '',
                avatars: [],
                imagesArray: [],
                typeArray: [],
                arr: [],
                obj: [],
                progress: 0
            }
        },
        computed: {
            loadedUser () {
                return this.$store.getters['users/loadedUser']
            },
            loadedAvatars () {
                return this.avatars.filter(avatar => avatar.gender === this.gender && avatar.type === this.bodyPart)
            },
            loadedPosts() {
                return this.$store.getters['posts/loadedPosts']
            }
        },
        methods: {
            selectGender(gender) {
                console.log(gender)
                this.gender = gender
                if (gender === 'female') {
                    for (let i = 0; i < this.obj.length; i++) {
                        this.obj[i].image = this.obj[i].image.replace('male', 'female')
                        this.obj[i].gender = 'female'
                    }
                    this.mergeImages()
                } else {
                    for (let i = 0; i < this.obj.length; i++) {
                        this.obj[i].image = this.obj[i].image.replace('female', 'male')
                        this.obj[i].gender = 'male'
                    }
                    this.mergeImages()
                }
            },
            selectBodyPart(part) {
                console.log(part)
                this.bodyPart = part
            },
            addToMerge(gender, type, image, name) {
                console.log('addToMerge')
                // console.log(gender)
                // console.log(type)
                // console.log(image)
                // console.log(name)
                this.name = name
                // if (name) {
                if (name.includes('background')) {
                    this.background = name
                } else if (name.includes('body')) {
                    this.body = name
                } else if (name.includes('skin')) {
                    this.skin = name
                } else if (name.includes('face')) {
                    this.face = name
                } else if (name.includes('hair')) {
                    this.hair = name
                }
                // }
                // Remove any image of the same type
                // if (this.arr.includes(image) || this.obj.includes(type)) {
                //     const index = this.arr.indexOf(image)
                //     this.arr.splice(index, 1)
                //     const index2 = this.obj.indexOf(type)
                //     this.obj.splice(index2, 1)
                // } else { // Add image if not present
                //     // this.arr.push(path)
                //     if (image == '/images/body.png') {
                //         this.arr.unshift(image)
                //     } else {
                //         this.arr.push(image)
                //         this.obj.push({'image': image, 'type': type})
                //     }
                // }
                // mergeImages()

                // Remove any existing image of the same type
                let found = false;
                for (let i = 0; i < this.obj.length; i++) {
                    if (this.obj[i].type === type) {
                        const index = this.obj.indexOf(this.obj[i])
                        this.obj.splice(index, 1)
                        found = true;
                        break;
                    }
                }

                // Add image to the object array
                // this.obj.push({'image': '/images/avatars/' + gender + '/' + type + '/' + image, 'gender': gender, 'type': type})
                if (type === 'background') {
                    this.obj.splice(0, 0, {'image': '/images/avatars/' + gender + '/' + type + '/' + image, 'gender': gender, 'type': type})
                } else if (type === 'face') {
                    this.obj.splice(4, 0, {'image': '/images/avatars/' + gender + '/' + type + '/' + image, 'gender': gender, 'type': type})
                } else if (type === 'hair') {
                    this.obj.splice(3, 0, {'image': '/images/avatars/' + gender + '/' + type + '/' + image, 'gender': gender, 'type': type})
                } else {
                    this.obj.splice(1, 0, {'image': '/images/avatars/' + gender + '/' + type + '/' + image, 'gender': gender, 'type': type})
                }
                this.mergeImages()
            },
            mergeImages() {
                // console.log(type)
                // console.log('obj: ')
                // console.log(this.obj)
                // Build an array of image paths from the object
                this.arr = []
                // this.arr = this.obj
                for (let i = 0; i < this.obj.length; i++) {
                    this.arr.push(this.obj[i].image)
                    // // this.arr.splice(i, 0, this.obj[i].image)
                    // if (this.obj[i].type === 'background') {
                    //     // this.arr.unshift(this.obj[i].image)
                    //     this.arr.splice(0, 0, this.obj[i].image)
                    // } else if (this.obj[i].type === 'face') {
                    //     this.arr.splice(1, 0, this.obj[i].image)
                    // } else {
                    //     this.arr.push(this.obj[i].image)
                    // }
                }
                // if (type === 'background') {
                //     this.arr.splice(0, 0, this.obj[0])
                // } else if (type === 'face') {

                // } else {

                // }

                if (process.browser) {
                    mergeImages(this.arr)
                    .then(
                        b64 => this.$refs.mergedImage.src = b64
                    )
                }
            },
            saveImage() {
                console.log('saveImage')
                // this.$store.dispatch('setLoading', true, { root: true })
                this.loading = true

                // Save image in Firebase Cloud Storage
                const now = moment().unix()
                const userId = firebase.auth().currentUser.uid
                const image_name = userId + '_' + this.gender + '_' + this.background + '_' + this.body + '_' + this.skin + '_' + this.face + '_' + this.hair
                console.log(image_name)
                let storageRef = firebase.storage().ref('/images/avatars/' + image_name)
                console.log(storageRef)
                let image = this.$refs.mergedImage.src

                var uploadTask = storageRef.putString(image, 'data_url')
                // return

                uploadTask.on('state_changed', (snapshot) => {
                    // Observe state change events such as progress, pause, and resume
                    // Get task progress, including the number of bytes uploaded and the total number of bytes to be uploaded
                    this.progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100
                    console.log('Upload is ' + this.progress + '% done')
                    switch (snapshot.state) {
                        case firebase.storage.TaskState.PAUSED: // or 'paused'
                        console.log('Upload is paused')
                        break
                        case firebase.storage.TaskState.RUNNING: // or 'running'
                        console.log('Upload is running')
                        break
                    }
                }, (error) => {
                  // Handle unsuccessful uploads
                  // this.$store.dispatch('setLoading', false, {root: true})
                  console.log(error)
                  this.loading = false
                // }, function() {
                }, () => {
                    // Handle successful uploads on complete
                    console.log(uploadTask.snapshot)
                    // const newImageKey = firebase.database().ref().child('/avatar_images/').push().key
                    firebase.database().ref('/users/' + userId + '/avatar').set({
                        name: uploadTask.snapshot.metadata.name,
                        url: uploadTask.snapshot.downloadURL,
                        updated_at: now,
                    })
                    console.log('Uploaded a data_url string!')
                    // this.$store.dispatch('setLoading', false, { root: true })
                    this.loading = false
                    new Noty({type: 'success', text: 'Successfully uploaded image!', timeout: 5000, theme: 'metroui'}).show()
                })
            }
        }
    }
</script>

<style scoped>
    .active {
        background-color: orangered;
    }
</style>
