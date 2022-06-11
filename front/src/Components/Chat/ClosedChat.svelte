<script lang="ts">
    import { fly } from "svelte/transition";
    import { chatMessagesStore, chatVisibilityStore } from "../../Stores/ChatStore";
    import ChatMessageForm from "./ChatMessageForm.svelte";
    import ChatElement from "./ChatElement.svelte";
    import { HtmlUtils } from "../../WebRtc/HtmlUtils";
    import LL from "../../i18n/i18n-svelte";
    import { localUserStore } from "../../Connexion/LocalUserStore";

    function showChat() {
        chatVisibilityStore.set(true);
    }
</script>

<aside class="chatWindow closed" on:click={showChat} transition:fly={{ y: -1000, duration: 500 }}>
    <section class="textForm">
        <div class="leftSide">
            <div class="chatImage" />
        </div>
        <div class="rightSide">
            <i class="arrow" />
            <span class="chatText">Chat</span>
            <br />
            <span>Discuter avec vos contacts </span>
        </div>
    </section>
</aside>

<style lang="scss">
    aside.chatWindow.closed {
        cursor: pointer;
        z-index: 1000;
        pointer-events: auto;
        position: absolute;
        bottom: 0;
        left: 0;
        height: 10vh;
        width: 25vw;
        max-width: 22vw;
        background: rgb(5, 31, 51, 0.9);
        color: whitesmoke;
        display: flex;
        flex-direction: column;
        max-height: 90vh;
        padding: 10px;
        border-bottom-right-radius: 16px;
        border-top-right-radius: 16px;

        .textForm {
            cursor: pointer;
            display: flex;
        }
        .rightSide {
            width: 200px;
            float: right;
            margin-left: 25px;
            .chatText {
                font-size: 25px;
                text-transform: uppercase;
            }
            .arrow {
                border: solid white;
                border-width: 0 3px 3px 0;
                display: inline-block;
                padding: 6px;
                transform: rotate(-135deg);
                -webkit-transform: rotate(-135deg);
                position: absolute;
                right: 120px;
                top: 4px;
                width: 16px;
                z-index: 111111;
            }
            span {
                font-weight: bold;
                font-size: 11px;
            }
        }
        .leftSide {
            width: 100px;
            height: 50px;
            border: 50px solid white;
            border-radius: 50px;
            margin-top: -32px;
            .chatImage {
                display: inline-block;
                width: 89px;
                height: 85px;
                background: url(resources/chat/traductor.png);
                background-size: cover;
                position: absolute;
                top: -21px;
                left: 16px;
            }
        }
    }
</style>
