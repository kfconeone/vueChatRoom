<template>
    <div id="chat" class="container">
        <div
            class="row justify-content-end fixed-bottom"
            style="padding-right: 10px; padding-bottom: 20px"
        >
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
                    @onClose="onClose"
                />
            </div>
        </div>
        <div
            v-if="!visible"
            class="row justify-content-end fixed-bottom"
            style="padding-right: 30px; padding-bottom: 10px"
        >
            <img src="../assets/chatIcon.png" @click="onRoomOpen()" />
        </div>
    </div>
</template>

<script>
import firebase from "firebase";
import dateFormat from "dateformat";
import { Chat } from "vue-quick-chat";
import "vue-quick-chat/dist/vue-quick-chat.css";

export default {
    name: "Home",
    components: { Chat },
    data() {
        return {
            //====聊天室元素====
            visible: false,
            participants: [
                {
                    name: "客服人員",
                    id: 1,
                },
            ],
            myself: {
                name: "我",
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
            chatTitle: "客服系統",
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
            this.registerRoom();
        }
    },
    methods: {
        onRoomOpen() {
            this.visible = true;
        },
        onMessageSubmit: function (message) {
            if (!this.$cookies.isKey("user_session")) {
                this.mychatroomRef = this.rootRef.push();
                this.$cookies.set("user_session", this.mychatroomRef.key);
                this.mychatroomRef.update({
                    updateDatetime: dateFormat(
                        new Date(),
                        "yyyy-mm-dd HH:MM:ss"
                    ),
                    isRead: false,
                });

                this.registerRoom();
            }

            this.mychatroomRef.update({ isRead: false });
            this.mychatroomRef
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
        registerRoom() {
            let self = this;
            this.mychatroomRef
                .child("messages")
                .on("child_added", function (snapshot) {
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

                    self.messages.push(msg);
                });
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
