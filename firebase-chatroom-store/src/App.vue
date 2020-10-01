<template>
    <div id="app" class="container-fluid">
        <div
            class="row"
            v-for="(room, index) in rooms"
            :key="room.id"
            style="margin-top: 10px"
        >
            <button
                type="button"
                class="btn"
                :class="room.isRead ? 'btn-light' : 'btn-danger'"
                @click="setChatroom(room.id)"
            >
                客戶{{ index }}
            </button>
        </div>

        <div class="row justify-content-end fixed-bottom">
            <div class="col chatroom">
                <Chat
                    id="chat-window"
                    v-if="visible"
                    :participants="participants"
                    :myself="myself"
                    :messages="messages"
                    :chat-title="chatTitle"
                    :placeholder="placeholder"
                    :profilePictureConfig="profilePictureConfig"
                    :colors="colors"
                    :scroll-bottom="scrollBottom"
                    :display-header="true"
                    :send-images="false"
                    @onMessageSubmit="onMessageSubmit"
                    @onType="onType"
                    @onClose="onClose"
                />
            </div>
        </div>
    </div>
</template>

<script>
import firebase from "firebase";
import dateFormat from "dateformat";
import { Chat } from "vue-quick-chat";
import "vue-quick-chat/dist/vue-quick-chat.css";

export default {
    name: "App",
    components: { Chat },
    data() {
        return {
            //====聊天室元素====
            visible: false,
            participants: [
                {
                    name: "客戶",
                    id: 1,
                },
            ],
            myself: {
                name: "客服人員",
                id: 2,
            },
            messages: [
                // {
                //     content: "received messages",
                //     myself: false,
                //     participantId: 1,
                //     timestamp: { year: 2019, month: 3, day: 5, hour: 20, minute: 10, second: 3, millisecond: 123 },
                //     type: "text",
                // },
                // {
                //     content: "sent messages",
                //     myself: true,
                //     participantId: 2,
                //     timestamp: { year: 2019, month: 4, day: 5, hour: 19, minute: 10, second: 3, millisecond: 123 },
                //     type: "text",
                // },
            ],
            chatTitle: "客服",
            placeholder: "請輸入訊息",
            colors: {
                header: {
                    bg: "#d30303",
                    text: "#fff",
                },
                message: {
                    myself: {
                        bg: "#fff",
                        text: "#bdb8b8",
                    },
                    others: {
                        bg: "#fb4141",
                        text: "#fff",
                    },
                    messagesDisplay: {
                        bg: "#f7f3f3",
                    },
                },
                submitIcon: "#b91010",
                submitImageIcon: "#b91010",
            },
            borderStyle: {
                topLeft: "10px",
                topRight: "10px",
                bottomLeft: "10px",
                bottomRight: "10px",
            },

            scrollBottom: {
                messageSent: true,
                messageReceived: false,
            },
            displayHeader: true,
            profilePictureConfig: {
                others: false,
                myself: false,
                styles: {
                    width: "30px",
                    height: "30px",
                    borderRadius: "50%",
                },
            },
            //頁面元素
            messageText: "",
            rooms: [],

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
            currentChatroomRef: null,
        };
    },
    created() {
        // Initialize Firebase
        firebase.initializeApp(this.firebaseConfig);
        this.database = firebase.database();
        this.rootRef = this.database.ref("chatroom");
        let self = this;

        this.rootRef.once("value", function (snapshot) {
            snapshot.forEach(function (childSnapshot) {
                let timespan = new Date(
                    new Date() -
                        new Date(Date.parse(childSnapshot.val().updateDatetime))
                );
                console.log(timespan.getHours());

                if (timespan.getHours() > 24) {
                    self.rootRef.child(childSnapshot.key).remove();
                }
            });
        });

        this.rootRef.on("value", function (snapshot) {
            let tempRoom = [];
            snapshot.forEach(function (childSnapshot) {
                tempRoom.push({
                    id: childSnapshot.key,
                    isRead: childSnapshot.val().isRead,
                });

                if (childSnapshot.val().isRead == false) {
                    window.document.getElementById("link-myIcon").href =
                        "/StorePage/favicon2.gif";
                }
            });
            self.rooms = tempRoom;
        });

        // if (this.$cookies.isKey("user_session")) {
        //     this.currentChatroomRef = this.rootRef.child(this.$cookies.get("user_session"));
        // } else {
        //     this.currentChatroomRef = this.rootRef.push();
        //     this.$cookies.set("user_session", this.currentChatroomRef.key, 10);
        // }

        // this.currentChatroomRef.set({
        //     updateDatetime: dateFormat(new Date(), "yyyy-mm-dd HH:MM:ss"),
        // });
        // this.currentChatroomRef.child("messages").on("child_added", (snapshot) => this.messages.push(snapshot.val()));
        // this.$cookies.set("user_session", "25j_7Sl6xDq2Kc3ym0fmrSSk2xV2XkUkX", 10);
        // console.log(this.$cookies.get("user_session"));
        // this.$cookies.remove("user_session");
        // console.log(this.$cookies.keys());
    },
    methods: {
        setChatroom(id) {
            console.log(id);
            window.document.getElementById("link-myIcon").href =
                "/StorePage/favicon.ico";
            let self = this;
            self.messages.length = 0;
            this.currentChatroomRef = this.rootRef.child(id);

            this.currentChatroomRef.update({
                updateDatetime: dateFormat(new Date(), "yyyy-mm-dd HH:MM:ss"),
                isRead: true,
            });

            this.currentChatroomRef
                .child("messages")
                .on("child_added", (snapshot) => {
                    let msg = {};
                    msg.content = snapshot.val().content;
                    msg.type = "text";
                    let time = new Date(Date.parse(snapshot.val().datetime));
                    msg.timestamp = {
                        year: time.getFullYear(),
                        month: time.getMonth(),
                        day: time.getDate(),
                        hour: time.getHours(),
                        minute: time.getMinutes(),
                        second: time.getSeconds(),
                        millisecond: time.getMilliseconds(),
                    };
                    if (snapshot.val().nickname == self.myself.name) {
                        msg.participantId = 2;
                        msg.myself = true;
                    } else {
                        msg.participantId = 1;
                        msg.myself = false;
                    }

                    // {
                    //     content: "received messages",
                    //     myself: false,
                    //     participantId: 1,
                    //     timestamp: { year: 2019, month: 3, day: 5, hour: 20, minute: 10, second: 3, millisecond: 123 },
                    //     type: "text",
                    // }

                    self.messages.push(msg);
                });
            this.visible = true;
        },
        onType: function (event) {
            console.log(event);
        },
        onMessageSubmit: function (message) {
            this.currentChatroomRef
                .child("messages")
                .push()
                .set({
                    datetime: dateFormat(new Date(), "yyyy-mm-dd HH:MM:ss"),
                    nickname: this.myself.name,
                    content: message.content,
                });
            setTimeout(() => {
                message.uploaded = true;
            }, 2000);
        },
        onClose() {
            this.visible = false;
        },
    },
};
</script>

<style scoped>
#chat-window {
    max-height: 600px;
    min-height: 400px;
    max-width: 400px;
}

.col.chatroom {
    max-width: 400px;
}
</style>
