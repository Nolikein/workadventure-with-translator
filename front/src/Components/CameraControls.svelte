<script lang="ts">
    import { requestedScreenSharingState, screenSharingAvailableStore } from "../Stores/ScreenSharingStore";
    import { requestedCameraState, requestedMicrophoneState, silentStore } from "../Stores/MediaStore";
    import monitorImg from "./images/monitor.svg";
    import monitorCloseImg from "./images/monitor-close.svg";
    import cinemaImg from "./images/cinema.svg";
    import cinemaCloseImg from "./images/cinema-close.svg";
    import microphoneImg from "./images/microphone.svg";
    import microphoneCloseImg from "./images/microphone-close.svg";
    import layoutPresentationImg from "./images/layout-presentation.svg";
    import layoutChatImg from "./images/layout-chat.svg";
    import followImg from "./images/follow.svg";
    import lockImg from "./images/lock.svg";
    import { LayoutMode } from "../WebRtc/LayoutManager";
    import { peerStore } from "../Stores/PeerStore";
    import { embedScreenLayoutStore } from "../Stores/EmbedScreensStore";
    import { followRoleStore, followStateStore, followUsersStore } from "../Stores/FollowStore";
    import { gameManager } from "../Phaser/Game/GameManager";
    import { currentPlayerGroupLockStateStore } from "../Stores/CurrentPlayerGroupStore";
    import { analyticsClient } from "../Administration/AnalyticsClient";

    const gameScene = gameManager.getCurrentGameScene();

    function screenSharingClick(): void {
        if ($silentStore) return;
        if ($requestedScreenSharingState === true) {
            requestedScreenSharingState.disableScreenSharing();
        } else {
            requestedScreenSharingState.enableScreenSharing();
        }
    }

    function cameraClick(): void {
        if ($silentStore) return;
        if ($requestedCameraState === true) {
            requestedCameraState.disableWebcam();
        } else {
            requestedCameraState.enableWebcam();
        }
    }

    function microphoneClick(): void {
        if ($silentStore) return;
        if ($requestedMicrophoneState === true) {
            requestedMicrophoneState.disableMicrophone();
        } else {
            requestedMicrophoneState.enableMicrophone();
        }
    }

    function switchLayoutMode() {
        if ($embedScreenLayoutStore === LayoutMode.Presentation) {
            $embedScreenLayoutStore = LayoutMode.VideoChat;
        } else {
            $embedScreenLayoutStore = LayoutMode.Presentation;
        }
    }

    function followClick() {
        switch ($followStateStore) {
            case "off":
                gameScene.connection?.emitFollowRequest();
                followRoleStore.set("leader");
                followStateStore.set("active");
                break;
            case "requesting":
            case "active":
            case "ending":
                gameScene.connection?.emitFollowAbort();
                followUsersStore.stopFollowing();
                break;
        }
    }

    function lockClick() {
        gameScene.connection?.emitLockGroup(!$currentPlayerGroupLockStateStore);
    }
</script>

<div class="btn-cam-action">
    <div class="btn-layout" on:click={switchLayoutMode} class:hide={$peerStore.size === 0} data-title="layout">
        {#if $embedScreenLayoutStore === LayoutMode.Presentation}
            <img class="noselect" src={layoutPresentationImg} style="padding: 2px" alt="Switch to mosaic mode" />
        {:else}
            <img class="noselect" src={layoutChatImg} style="padding: 2px" alt="Switch to presentation mode" />
        {/if}
    </div>

    <div
        class="btn-follow"
        class:hide={($peerStore.size === 0 && $followStateStore === "off") || $silentStore}
        class:disabled={$followStateStore !== "off"}
        on:click={() => analyticsClient.follow()}
        on:click={followClick}
        data-title="Suivre"
    >
        <img class="noselect" src={followImg} alt="" />
    </div>

    <div
        class="btn-lock"
        class:hide={$peerStore.size === 0 || $silentStore}
        class:disabled={$currentPlayerGroupLockStateStore}
        on:click={() => analyticsClient.lockDiscussion()}
        on:click={lockClick}
        data-title="Verrouiller"
    >
        <img class="noselect" src={lockImg} alt="" />
    </div>

    <div
        class="btn-monitor"
        on:click={() => analyticsClient.screenSharing()}
        on:click={screenSharingClick}
        class:hide={!$screenSharingAvailableStore || $silentStore}
        class:enabled={$requestedScreenSharingState}
        data-title="Partage ton ecran"
    >
        {#if $requestedScreenSharingState && !$silentStore}
            <img class="noselect" src={monitorImg} alt="Start screen sharing" />
        {:else}
            <img class="noselect" src={monitorCloseImg} alt="Stop screen sharing" />
        {/if}
    </div>

    <div
        class="btn-video"
        on:click={() => analyticsClient.camera()}
        on:click={cameraClick}
        class:disabled={!$requestedCameraState || $silentStore}
        data-title="Camera"
    >
        {#if $requestedCameraState && !$silentStore}
            <img class="noselect" src={cinemaImg} alt="Turn on webcam" />
        {:else}
            <img class="noselect" src={cinemaCloseImg} alt="Turn off webcam" />
        {/if}
    </div>

    <div
        class="btn-micro"
        on:click={() => analyticsClient.microphone()}
        on:click={microphoneClick}
        class:disabled={!$requestedMicrophoneState || $silentStore}
        data-title="Audio"
    >
        {#if $requestedMicrophoneState && !$silentStore}
            <img class="noselect" src={microphoneImg} alt="Turn on microphone" />
        {:else}
            <img class="noselect" src={microphoneCloseImg} alt="Turn off microphone" />
        {/if}
    </div>
</div>

<style lang="scss">
    @import "../../style/breakpoints.scss";

    div[data-title] {
        position: relative;
        transition: opacity 1s;
    }

    div[data-title]::after {
        content: attr(data-title);
        position: absolute;
        left: 90%;
        top: 20%;
        transform: translate(-50%,-150%);
        background: #e5e7eb;
        color: #18314b;
        border-radius: 0.4rem;
        padding: .115rem .875rem;
        white-space: nowrap;
        opacity: 0;
        transition: opacity 0.5s;
    }

    div[data-title]:hover::after {
        opacity: 1;
    }

    .btn-cam-action {
        pointer-events: all;
        position: absolute;
        display: inline-flex;
        bottom: 10px;
        right: 15px;
        width: 360px;
        height: 40px;
        text-align: center;
        align-content: center;
        justify-content: flex-end;
        z-index: 251;

        &:hover {
            div.hide {
                transform: translateY(60px);
            }
        }
    }
    /*btn animation*/
    .btn-cam-action div {
        cursor: url("../../style/images/cursor_pointer.png"), pointer;
        display: flex;
        align-items: center;
        justify-content: center;
        border: solid 0px black;
        width: 44px;
        height: 44px;
        background: #666;
        box-shadow: 2px 2px 24px #444;
        border-radius: 48px;
        transform: translateY(15px);
        transition-timing-function: ease-in-out;
        transition: all 0.3s;
        margin: 0 2%;

        &.hide {
            transform: translateY(60px);
        }
    }
    .btn-cam-action div.disabled {
        background: #d75555;
    }
    .btn-cam-action div.enabled {
        background: #73c973;
    }
    .btn-cam-action:hover div {
        transform: translateY(0);
    }
    .btn-cam-action div:hover {
        background: #407cf7;
        box-shadow: 4px 4px 48px #666;
        transition: 120ms;
    }
    .btn-micro {
        pointer-events: auto;
    }
    .btn-video {
        pointer-events: auto;
        transition: all 0.25s;
    }
    .btn-monitor {
        pointer-events: auto;
    }
    .btn-layout {
        pointer-events: auto;
        transition: all 0.15s;
    }
    .btn-cam-action div img {
        height: 22px;
        width: 30px;
        position: relative;
        cursor: url("../../style/images/cursor_pointer.png"), pointer;
    }

    .btn-follow {
        pointer-events: auto;

        img {
            filter: brightness(0) invert(1);
        }
    }

    .btn-lock {
        pointer-events: auto;

        img {
            filter: brightness(0) invert(1);
        }
    }

    @media (hover: none) {
        /**
        * If we cannot hover over elements, let's display camera button in full.
        */
        .btn-cam-action {
            div {
                transform: translateY(0px);
            }
        }
    }

    @include media-breakpoint-up(sm) {
        .btn-cam-action {
            right: 0;
            width: 100%;
            height: 40%;
            max-height: 40px;

            div {
                width: 20%;
                max-height: 44px;
            }
        }
    }
</style>
