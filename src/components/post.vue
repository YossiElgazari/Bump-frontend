<template>
    <div class="post-container">

        <div class="poststart">
            <div v-if="mine" class="deletebutton" @click="showdelete = !showdelete">
                <img v-if="!deletehoverpost" src="../assets/delete.svg" alt="Delete Icon"
                    @mouseenter="deletehoverpost = true" />
                <img v-else src="../assets/deleteorange.svg" alt="Delete Icon" @mouseout="deletehoverpost = false" />
            </div>
            <div v-if="mine && !post.isShared" class="editbutton" @click="showedit = !showedit">
                <img v-if="!edithoverpost" src="../assets/edit_square.svg" style="width: 18px;" alt="edit Icon"
                    @mouseenter="edithoverpost = true" />
                <img v-else src="../assets/edit_square_hover.svg" alt="edit Icon" style="width: 23px; margin-top:1px"
                    @mouseout="edithoverpost = false" />
            </div>
            <div class="profile-picture">
                <img :class="{ mineclass: mine }" :src="post.userProfilePicture" @click="movetoprofile(post.userID)"
                    alt="Profile Picture" />
            </div>
            <div class="post-content">

                <div class="user-info">
                    <div class="user-name" @click="movetoprofile(post.userID)">{{ post.GamerTag }}</div>
                    <div class="post-date">{{ date(post.date) }}</div>

                </div>
                <div v-if="!post.isShared && !showedit" class="post-text" style="white-space: pre-line">
                    {{ post.text }}
                </div>

                <div v-if="showedit && !post.isShared">
                    <textarea v-model="inputText" :rows="textareaRows" :cols="textareaCols" maxlength="1024"
                        placeholder="Edit your post here."></textarea>
                    <button class="btn" @click="submitPost">Post</button>
                </div>
                <div v-if="post.isShared" class="sharedpost">
                    <div class="profile-picture">
                        <img :class="{ mineclass: post.GamerTag === post.SGamerTag }" :src="post.Spicture"
                            alt="Profile Picture" @click="movetoprofile(post.Suserid)" />
                    </div>

                    <div class="post-content">

                        <div class="user-info">
                            <div class="user-name" @click="movetoprofile(post.Suserid)">{{ post.SGamerTag }}</div>
                            <div class="post-date">{{ date(post.Sdate) }}</div>
                        </div>
                        <div class="post-text" style="white-space: pre-line">
                            {{ post.text }}

                        </div>
                    </div>

                </div>
            </div>

        </div>
        <div>
            <div class="allthepost">
                <div class="restofpost">

                    <div class="post-photo" v-if="post.picture != 'null'">

                        <img :src="post.picture" alt="Post Photo" />

                    </div>

                    <div class="post-actions">
                        <div class="bump">
                            <img src="../assets/bump.svg" alt="Bump Icon" style="width: 22px; margin:0;" />
                            <span class="bump-count">{{ post.numOfBumps }} Bumps</span>
                        </div>
                        <div v-if="post.picture === 'null' && !post.isShared" class="share">
                            <img src="../assets/reply.svg" alt="Share Icon" />
                            <span class="share-count">{{ post.numofshares }} Shares</span>
                        </div>
                    </div>
                    <div class="checkboxes">
                        <div class="checkbox checkbump" @click="pressonbump">
                            <img v-if="!bumpselected" src="../assets/bump.svg" alt="Bump Icon" style="width: 25px;" />
                            <img v-else src="../assets/bumpselected.svg" alt="Bump Icon" style="width: 25px;" />
                            <p :class="{ selected: bumpselected }"> Bump</p>
                        </div>
                        <div class="outshare">
                            <div class="checkbox checkshare" v-if="post.picture === 'null' && !post.isShared"
                                @click="pressonshare">
                                <img src="../assets/reply.svg" alt="Share Icon" style="width: 18px;" />
                                <p> Share</p>
                            </div>
                        </div>
                        <div v-if="!post.isShared" class="checkbox checksave" @click="pressonsave">
                            <p :class="{ selected: saveselected }"> Save</p>
                            <img v-if="!saveselected" src="../assets/save.svg" alt="Save Icon" style="width: 25px;" />
                            <img v-else src="../assets/saveselected.svg" alt="Save Icon" style="width: 25px;" />
                        </div>
                    </div>
                    <div class="comments">
                        <div class="comment-section">
                            <div class="comment-profile-picture_mine">
                                <img class="profilepict" :src="profilePicture" alt="User Profile Picture" />
                            </div>
                            <div class="comment-input">
                                <input type="text" v-model="mycomment" @keyup.enter="sendcomment"
                                    placeholder="Write a comment here..." />
                                <img v-if="!sendhover" class="sendicon" src="../assets/send.svg" alt="Comment Icon"
                                    @mouseenter="sendhover = true" @click="sendcomment">
                                <img v-else class="sendicon" src="../assets/sendhover.svg" alt="Comment Icon"
                                    @mouseout="sendhover = false" @click="sendcomment" />
                            </div>
                        </div>
                        <div v-if="visibleComments != []" class="comment" v-for="(comment, index) in visibleComments"
                            :key="comment.id" @mouseenter="commenthover = index" @mouseleave="deletehover = false">
                            <div class="comment-profile-picture" v-if="comment.Picture != 'null'">
                                <img :class="{ mineclass: comment.userID === userId }" :src="comment.Picture"
                                    alt="User Profile Picture" @click="movetoprofile(comment.userID)" />
                            </div>

                            <div class="comment-content">
                                <div class="user-info">

                                    <div class="user-name" @click="movetoprofile(comment.userID)">{{ comment.GamerTag }}
                                    </div>
                                </div>
                                <div class="comment-text">
                                    {{ comment.text }}
                                </div>
                                <div v-if="(post.userID === userId && commenthover === index) || (comment.userID === userId && commenthover === index)"
                                    class="deletebuttoncomment" @mouseenter="deletehover = true">
                                    <img v-if="!deletehover" src="../assets/delete.svg" alt="Delete Icon" />
                                    <img v-else src="../assets/deleteorange.svg" alt="Delete Icon" />
                                    <div v-if="deletehover" class="delete-options">
                                        <span>Delete?</span>
                                        <div class="delete-option">
                                            <img v-if="!cancelhover" id="cancelIcon" src="../assets/cancel.svg"
                                                alt="Cancel Icon" @click="deletehover = false"
                                                @mouseenter="cancelhover = true" />
                                            <img v-else id="cancelIcon" @click="deletehover = false"
                                                src="../assets/cancelgreen.svg" alt="Cancel Icon"
                                                @mouseleave="cancelhover = false" />
                                            <img v-if="!checkhover" @click="deletecomment" id="checkIcon"
                                                src="../assets/check_circle.svg" alt="Check Circle Icon"
                                                @mouseenter="checkhover = true" />
                                            <img v-else id="checkIcon" @click="deletecomment"
                                                src="../assets/check_circle_green.svg" alt="Check Circle Icon"
                                                @mouseleave="checkhover = false" />
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div v-if="post.comments.length > 3">
                            <button v-if="showAllComments" class="allcomments" @click="toggleComments">View Less
                                Comments</button>
                            <button class="allcomments" v-else @click="toggleComments">View More Comments</button>
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </div>

    <div v-if="showdelete">
        <deletecomp @closeA="colosedelete" />
    </div>

    <loading v-if="isloading" />
</template>
  
<script>
import axios from 'axios';
import deletecomp from '../components/delete.vue';
import loading from './loading.vue';

export default {
    name: 'post',
    components: {
        deletecomp, loading
    },
    props: ['post', 'profilePicture', 'GamerTag', 'myGamerTag'],
    data() {
        return {
            mycomment: '',
            bumpselected: this.post.hasUserBumped,
            saveselected: this.post.hasUserSaved,
            userId: this.$route.query.id,
            sendhover: false,
            showAllComments: false,
            visibleComments: this.post.comments,
            mine: false,
            deletehover: false,
            showdelete: false,
            deleteornot: false,
            commenthover: false,
            cancelhover: false,
            checkhover: false,
            deletehoverpost: false,
            edithoverpost: false,
            showedit: false,
            inputText: this.post.text,
            textareaRows: 1,
            textareaCols: 20,
            isloading: false,
        };
    },
    watch: {
        inputText() {
            this.adjustTextareaSize();
        }
    },
    created() {
        this.updateVisibleComments();
        this.userId = this.$route.query.id;
        console.log("userId:" + this.userId);
        console.log("post.userId:" + this.post.userID);
        console.log("post.Sid:" + this.post.Sid);
        console.log("post:" + this.post);
        if (this.post.userID === this.userId) {
            this.mine = true;
        }
        this.bumpselected = this.post.hasUserBumped
        this.saveselected = this.post.hasUserSaved

    }, methods: {
        adjustTextareaSize() {
            const lines = this.inputText.split('\n');
            this.textareaRows = lines.length;
            this.textareaCols = Math.max(...lines.map(line => line.length)) + 1;
        },
        pressonsave() {
            if (!this.saveselected && !this.post.isShared) {
                var addr = 'https://backend-project-vzn7.onrender.com/savepost';
            }
            else if (this.saveselected && !this.post.isShared) {
                var addr = 'https://backend-project-vzn7.onrender.com/removesaved';
            }
            this.saveselected = !this.saveselected
            const objecttopass = {
                "user": this.userId,
                "post": this.post._id
            }

            const res = this.requestfromserver(addr, objecttopass)
            if (!res) {
                console.log("save failed");
            }

        },
        submitPost() {
            if (this.inputText === '') {
                return;
            }
            this.isloading = true;
            var addr = 'https://backend-project-vzn7.onrender.com/editpost';
            const objecttopass = {
                "postid": this.post._id,
                "txt": this.inputText
            }
            const res = this.requestfromserver(addr, objecttopass)
            if (res) {
                this.post.text = this.inputText;
                this.showedit = false;
            }
            else {
                console.log("edit failed");
            }
            this.isloading = false;
        },
        movetoprofile(id) {
            this.searchResults = [];
            this.$router.push({ name: 'profile', query: { id: this.userId }, params: { differentUserId: id } });
            this.$emit('openProfile', id);
        },
        sendcomment() {
            if (this.mycomment === '') {
                return;
            }
            var objecttopass = {};
            if (this.post.isShared) {
                var addr = 'https://backend-project-vzn7.onrender.com/addcommenttoshare';
            }
            else {
                var addr = 'https://backend-project-vzn7.onrender.com/addcomment';
            }
            objecttopass = {
                "user": this.userId,
                "post": this.post._id,
                "text": this.mycomment
            }
            const res = this.requestfromserver(addr, objecttopass)
            if (res) {
                console.log("this.GamerTag:", this.GamerTag);
                const newcomment = {
                    "_id": res,
                    "userID": this.userId,
                    "GamerTag": this.myGamerTag,
                    "text": this.mycomment,
                    "Picture": this.profilePicture,
                    "date": "2021-05-01T00:00:00.000Z"
                }
                this.mycomment = '';
                this.post.comments.unshift(newcomment);
                this.updateVisibleComments();
            }
            else {
                console.log("comment failed");
            }

        },
        async pressonshare() {
            var addr = 'https://backend-project-vzn7.onrender.com/sharepost';
            const objecttopass = {
                "user": this.userId,
                "post": this.post._id
            }
            console.log("pressonshare:", objecttopass);
            this.isloading = true;
            var res = await this.requestfromserver(addr, objecttopass)
            this.isloading = false;
            if (res) {
                this.$emit('deletepost');
            }
            else {
                console.log("share failed");
            }
        },
        async requestfromserver(addr, objecttopass) {
            console.log("addr:", addr);
            try {

                const response = await axios.post(addr, objecttopass);

                var res = response.data;
                console.log("res:", res);

                return res;
            } catch (error) {
                console.error(error);
            }
        },
        deletecomment() {
            console.log("deletecomment:" + this.post.comments[this.commenthover].userID);
            var objecttopass = {};
            if (this.post.isShared) {
                var addr = 'https://backend-project-vzn7.onrender.com/removecommentfromshare';
            }

            else {
                var addr = 'https://backend-project-vzn7.onrender.com/removecomment';
            }
            objecttopass = {
                "user": this.post.comments[this.commenthover].userID,
                "post": this.post._id,
                "comment": this.post.comments[this.commenthover]._id
            }
            console.log("deletecomment:", objecttopass);
            const res = this.requestfromserver(addr, objecttopass)
            if (res) {
                this.post.comments.splice(this.post.comments[this.commenthover]._id, 1);
                this.updateVisibleComments();
            }
            else {
                console.log("delete failed");
            }
        },
        async colosedelete(deleteornot) {
            this.showdelete = false;
            this.deleteornot = deleteornot;
            this.userID = this.$route.query.id;
            console.log("deleteornot:" + this.deleteornot);
            if (this.deleteornot) {
                var objecttopass = {};
                if (this.post.isShared) {
                    var addr = 'https://backend-project-vzn7.onrender.com/removeshare';
                    objecttopass = {
                        "user": this.userId,
                        "post": this.post.Sid,
                        "share": this.post._id,
                    }
                }
                else {
                    var addr = 'https://backend-project-vzn7.onrender.com/removepost';
                    objecttopass = {
                        "user": this.userId,
                        "post": this.post._id
                    }
                }
                var res = await this.requestfromserver(addr, objecttopass)
                console.log("res:", res);
                if (res) {
                    this.$emit('deletepost');
                }
                else {
                    console.log("delete failed");
                }
            }
        },
        toggleComments() {
            this.showAllComments = !this.showAllComments;
            this.updateVisibleComments();
        },
        updateVisibleComments() {
            if (this.showAllComments) {
                this.visibleComments = this.post.comments;
            } else {
                this.visibleComments = this.post.comments.slice(0, 3);
            }
        },
        pressonbump() {
            var objecttopass = {};
            if (!this.bumpselected && !this.post.isShared) {
                this.post.numOfBumps++;
                this.bumpselected = true;
                var addr = 'https://backend-project-vzn7.onrender.com/addbump';
            }
            else if (this.bumpselected && !this.post.isShared) {
                this.post.numOfBumps--;
                this.bumpselected = false;

                var addr = 'https://backend-project-vzn7.onrender.com/removebump';
            }
            else if (!this.bumpselected && this.post.isShared) {
                this.post.numOfBumps++;
                this.bumpselected = true;

                var addr = 'https://backend-project-vzn7.onrender.com/addbumptoshare';
            }
            else if (this.bumpselected && this.post.isShared) {
                this.post.numOfBumps--;
                this.bumpselected = false;

                var addr = 'https://backend-project-vzn7.onrender.com/removebumpfromshare';
            }

            objecttopass = {
                "user": this.userId,
                "post": this.post._id
            }

            const res = this.requestfromserver(addr, objecttopass)

            if (!res) {
                console.log("bump failed");
            }

        },
        date(date) {
            var date = new Date(date);
            const options = {
                year: 'numeric',
                month: '2-digit',
                day: '2-digit',
                hour: '2-digit',
                minute: '2-digit',
                timeZone: 'Asia/Jerusalem',
            };
            return date.toLocaleString('en-IL', options);
        },
    }
}


</script>

<style scoped>
.post-container {
    width: calc(100vw - 50vw);
    background-color: var(--background);
    position: relative;
    border: 1px solid var(--stroke);
    border-radius: 15px;
    margin-bottom: 30px;
    /* Add any desired styling for the post container */
}

.poststart {
    gap: 30px;
    padding: 20px;
    padding-right: 60px;
    display: flex;

}

.sharedpost {
    gap: 30px;
    display: flex;
}

.profile-picture img {
    width: 60px;
    height: 60px;
    object-fit: fill;
    cursor: pointer;
}



.profile-picture img {
    border-radius: 50%;
    border: 2px solid var(--white);
}

.user-info {
    display: flex;
    flex-direction: column;
    margin-bottom: 15px;
    margin-top: 10px;
}

.user-name {
    font-weight: bold;
    margin-right: 10px;
    color: var(--white);
}

.user-name:hover {
    cursor: pointer;
    color: var(--main);
}

.post-date {
    font-size: 14px;
    color: var(--grey);

}

.post-text {
    margin-bottom: 20px;
    color: var(--white);

}

.post-photo {
    position: relative;
    width: 100%;
    height: 600px;

    background-color: var(--thirdcolor);
    margin-bottom: 10px;
}

.post-photo img {
    position: absolute;
    width: 100%;
    height: 100%;
    object-fit: scale-down;
    top: 0;
    left: 0;
}

.post-actions {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
    padding: 0 20px;
}


.bump,
.share {
    display: flex;
    align-items: center;
    color: var(--gray);
}


.checkbox {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--gray);
}

.checkbox:hover {
    cursor: pointer;
    background-color: var(--stroke);
    border-radius: 5px;
}



.checkbox p {
    color: var(--grey);
}

.checkbox img {
    margin-right: 5px;
    width: 20px;
}

.checkbump {
    padding: 0 50px;
}

.checksave {
    padding: 0 50px;
}

.bump-count,
.share-count {
    margin-left: 5px;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    color: var(--grey);
    font-size: 14px;
}

.bump img {
    width: 25px;
    margin-right: 5px;
}

.share img {
    width: 15px;
    margin-right: 5px;
}

.checkboxes {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-around;
    margin-bottom: 10px;
    border-top: 1px solid var(--stroke);
    border-bottom: 1px solid var(--stroke);
    padding: 5px 10px;
}

.outshare {
    border-right: 1px solid var(--stroke);
    border-left: 1px solid var(--stroke);
    padding: 0 5px;
    margin: 0 5px;

}

.checkshare {
    width: auto;
    padding: 0 110px;

}

.checkshare img {
    width: 15px;

}

.checkbox-label {
    display: flex;
    align-items: center;
    margin-right: 20px;
    color: var(--white);
}

.checkbox-input {
    margin-right: 5px;
}

.checkbox-icon {
    width: 16px;
    height: 16px;
    margin-right: 5px;
}

.comment-section {
    display: flex;
    align-items: center;
    padding: 20px;

}

.comment-profile-picture img {
    border: 2px solid var(--white);
    width: 40px;
    height: 40px;
    border-radius: 50%;
    margin-right: 15px;
    margin-top: 7px;
}

.comment-input input {
    width: calc(100% - 90px);
    resize: vertical;
    border-radius: 7.5px;
    min-height: 25px;
    background-color: var(--thirdcolor);
    border: none;
    color: var(--white);
    padding-left: 15px;
    padding-right: 60px;
    height: 55px;

}

.comment-profile-picture_mine img {
    border: 2px solid var(--main);
    width: 50px;
    height: 50px;
    border-radius: 50%;
    margin-right: 10px;
    margin-top: 7px;
}

.comment-input input::placeholder {
    color: var(--white);
    opacity: 0.5;
}

.comment-input {
    position: relative;
    width: 100%;
}


.sendicon {
    position: absolute;
    cursor: pointer;
    top: 50%;
    right: 30px;
    transform: translate(-50%, -50%);
}

p.selected {
    color: var(--main);
    cursor: pointer;
    border-radius: 15px;
}


.comment {

    display: flex;
    align-items: flex-start;
    padding: 10px;
    padding-left: 30px;
    padding-right: 80px;
}



.comment-content {
    position: relative;
    display: flex;
    flex-direction: column;
    border-radius: 7.5px;
    min-height: 25px;
    max-width: calc(100% - 130px);
    background-color: var(--thirdcolor);
    border: none;
    color: var(--white);
    padding: 5px 15px;
    padding-bottom: 15px;
}

.comment-content .user-info {
    margin-bottom: 15px;
}

.comment-text {
    color: var(--white);
    overflow: hidden;
    word-wrap: break-word;

}

.allcomments {
    padding: 10px;
    padding-left: 30px;
    border: none;
    background-color: rgba(255, 255, 255, 0);
    border: none;
    color: var(--white);
    font-size: 16px;
    border-radius: 7.5px;
    margin-bottom: 10px;
}

.allcomments:hover {
    cursor: pointer;
    text-decoration: underline;
}

.mineclass {
    border: 2px solid var(--main) !important;
}

.deletebutton {
    position: absolute;
    cursor: pointer;
    top: 15px;
    right: 15px;
}

.editbutton {
    position: absolute;
    cursor: pointer;
    top: 14px;
    right: 50px;
}

textarea {
    width: 200%;
    min-height: 100px;
    padding: 10px 20px;
    border-radius: 15px;
    border: 2px solid var(--stroke);
    background-color: var(--thirdcolor);
    color: var(--white);
    font-family: var(--mainfont);
    font-size: 16px;
    font-weight: 400;
    letter-spacing: 0.5px;
    resize: vertical;
    overflow: scroll;
    word-wrap: break-word;
}


.deletebuttoncomment {
    display: flex;
    flex-direction: row;
    cursor: pointer;
    top: 38%;
    right: -38px;


}

.deletebuttoncomment img {
    position: absolute;
    top: 38%;
    right: -38px;
}

.delete-options {
    position: absolute;
    top: 18%;
    right: -98px;
    display: flex;
    flex-direction: column;
    transition: 0.3s;
}

.delete-option {
    position: absolute;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    width: 100px;
    transition: 0.3s;

}

#cancelIcon {
    position: absolute;
    right: 78px;
    top: 25px
}

#checkIcon {
    position: absolute;
    right: 48px;
    top: 25px
}

.btn {
    margin-top: 10px;
    border: 3px solid var(--white);
    background-color: var(--stroke);
    color: var(--main);
    padding: 10px 20px;
    font-weight: bold;
    font-size: 15px;
    border-radius: 7px;
    transition: 0.5s;
}

.btn:hover {
    background-color: var(--main);
    color: var(--white);
    cursor: pointer;
}
</style>
