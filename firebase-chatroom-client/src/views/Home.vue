<template>
    <div id="chat" class="container">
        <h1>Vue School Chat Room</h1>
        <!-- Messages -->
        <div v-for="(message, index) in messages" class="card" :key="index">
            <div class="card-body">
                <!-- nickname -->
                <h6 class="card-subtitle mb-2 text-muted">
                    {{ message.nickname }}
                </h6>
                <!-- content -->
                <p class="card-text">{{ message.text }}</p>
            </div>
        </div>

        <hr />
        <!-- New Message -->
        <form @submit.prevent="storeMessage">
            <div class="form-group">
                <label>Message:</label>
                <textarea v-model="messageText" class="form-control"></textarea>
            </div>
            <div class="form-group">
                <label>Nickname:</label>
                <input v-model="nickname" class="form-control" />
            </div>
            <button class="btn btn-primary">Send</button>
        </form>
    </div>
</template>

<script>
import firebase from "firebase";
import dateFormat from "dateformat";

export default {
    name: "Home",
    components: {},
    data() {
        return {
            //頁面元素
            messages: [],
            messageText: "",
            nickname: "hootlex",
            //db元素
            firebaseConfig: {
                apiKey: "AIzaSyBNhudq5zJ3SGEuZl7ZccpWwDo_hv1iwfk",
                authDomain: "testing-vue-bb3af.firebaseapp.com",
                databaseURL: "https://testing-vue-bb3af.firebaseio.com",
                projectId: "testing-vue-bb3af",
                storageBucket: "testing-vue-bb3af.appspot.com",
                messagingSenderId: "403045099754",
                appId: "1:403045099754:web:8c1339ef83167be9244f93",
                measurementId: "G-6F3QCKLWTM",
            },
            db: null,
            rootRef: null,
            mychatroomRef: null,
        };
    },
    created() {
        // Initialize Firebase
        firebase.initializeApp(this.firebaseConfig);
        this.database = firebase.database();
        this.rootRef = this.database.ref("chatroom");
        if (this.$cookies.isKey("user_session")) {
            this.mychatroomRef = this.rootRef.child(
                this.$cookies.get("user_session")
            );
        } else {
            this.mychatroomRef = this.rootRef.push();
            this.$cookies.set("user_session", this.mychatroomRef.key);
        }

        this.mychatroomRef.update({
            updateDatetime: dateFormat(new Date(), "yyyy-mm-dd HH:MM:ss"),
        });
        this.mychatroomRef
            .child("messages")
            .on("child_added", (snapshot) =>
                this.messages.push(snapshot.val())
            );
        // this.$cookies.set("user_session", "25j_7Sl6xDq2Kc3ym0fmrSSk2xV2XkUkX", 10);
        // console.log(this.$cookies.get("user_session"));
        // this.$cookies.remove("user_session");
        // console.log(this.$cookies.keys());
    },
    methods: {
        storeMessage() {
            this.$cookies.set("user_session", this.mychatroomRef.key);
            this.mychatroomRef.update({ isRead: false });
            this.mychatroomRef.child("messages").push({
                datetime: dateFormat(new Date(), "yyyy-mm-dd HH:MM:ss"),
                nickname: this.nickname,
                content: this.messageText,
            });

            // {
            //     content: "received messages",
            //     myself: false,
            //     participantId: 1,
            //     timestamp: { year: 2019, month: 3, day: 5, hour: 20, minute: 10, second: 3, millisecond: 123 },
            //     type: "text",
            // },
            this.messageText = "";
        },
    },
};
</script>
