<script setup>
import { USER_ID } from '@/constants/cookie';
import { useMessageStore } from '@/stores/messageStore'
import { MessageOutlined } from '@ant-design/icons-vue';
import dayjs from 'dayjs';
import Cookies from 'js-cookie'
import { watchEffect } from 'vue';

const messageStore = useMessageStore()

watchEffect(() => {
    if(messageStore.messageSocket) {
        messageStore.messageSocket.onmessage = function(event) {
            let receiveMesssage = JSON.parse(event.data)

            if(receiveMesssage.roomID == messageStore.roomID) {
                if (receiveMesssage.type == "typing") {
                    if (receiveMesssage.userID == messageStore.userConversationID) {
                        messageStore.isTyping = receiveMesssage.content == "0" ? false : true
                    }
                }

                if (receiveMesssage.type == "sent-message") {
                    if (receiveMesssage.userID == messageStore.userConversationID) {
                        let dataMessage = { 
                            message: receiveMesssage.content, 
                            userID: receiveMesssage.userID, 
                            time: receiveMesssage.time
                        }

                        messageStore.messages = messageStore.messages.concat(dataMessage)
                    }
                }
            }
          
        
        };
    }
})

const onTyping = () => {
    let userID = Cookies.get(USER_ID)

    if(messageStore.userConversationID && messageStore.messageSocket && messageStore.roomID) {
        messageStore.messageSocket.send(JSON.stringify({
            RoomID: messageStore.roomID,
            Type: "typing",
            UserID: userID,
            SenderID: messageStore.userConversationID,
            Content: "1",
        }))
    }
}

const onBlur = () => {
    let userID = Cookies.get(USER_ID)

    if(messageStore.userConversationID && messageStore.messageSocket && messageStore.roomID) {
        messageStore.messageSocket.send(JSON.stringify({
            RoomID: messageStore.roomID,
            Type: "typing",
            UserID: userID,
            SenderID: messageStore.userConversationID,
            Content:  "0",
        }))
    }
}

const onSendMessage = (e) => { 
    e.preventDefault();

    if(messageStore.messageText) {
        let userID = Cookies.get(USER_ID)
    
        if(messageStore.userConversationID && messageStore.messageSocket && messageStore.roomID) {
            messageStore.isScrollBottom = true
            messageStore.messageSocket.send(JSON.stringify({
                RoomID: messageStore.roomID,
                Type: "sent-message",
                UserID: userID,
                SenderID: messageStore.userConversationID,
                Content:  messageStore.messageText,
            }))

            messageStore.messages = messageStore.messages.concat({
                message: messageStore.messageText,
                userID: userID,
                time: dayjs(),
            })

            messageStore.messageText = ''
        }
    }
}
</script>


<template>
        <form 
            class="
                ml-0 border-t-2 flex items-center absolute px-2 shadow-lg w-full bottom-0 left-0 h-[60px] bg-[white]
                max-md:fixed max-md:z-[9999]
            "
            @submit="(e) => {
                onSendMessage(e)
            }"
        >
            <div 
                className='absolute flex justify-center items-center space-x-2 top-[-55%] bg-white py-1 px-3 rounded-lg text-sm'
                v-if="messageStore.isTyping"
            >
                <span className='text-[#9E9EA1]'>{{ messageStore.userConversation }}</span>
                <div className='typing-status'>
                    <span></span>
                    <span></span>
                    <span></span>
                </div>
            </div>
            <input
                v-if="messageStore.userConversationID"
                type="text"
                className="form-input pl-5 outline-none h-9 w-full bg-transparent placeholder:text-slate-400/70"
                :placeholder="`Nhắn gì đó cho ${messageStore.userConversation}`"
                v-model="messageStore.messageText"
                @focus="() => {
                    onTyping()
                }"
                @blur="() => {
                    onBlur()
                }"
            />
            <button
                v-if="messageStore.userConversationID"
                className="btn outline-none flex justify-center items-center h-9 w-9 shrink-0 rounded-full p-0 text-primary hover:bg-primary/20 focus:bg-primary/20 active:bg-primary/25"
                type='submit'
            >
                <MessageOutlined class="text-[#87d068] text-[25px]" />
            </button>
        </form>
</template>