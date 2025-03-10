<script lang="ts">
    import { highlightedEmbedScreen } from "../../../Stores/EmbedScreensStore";
    import CamerasContainer from "../CamerasContainer.svelte";
    import MediaBox from "../../Video/MediaBox.svelte";
    import { coWebsiteManager } from "../../../WebRtc/CoWebsiteManager";
    import { afterUpdate, onMount } from "svelte";
    import { isMediaBreakpointDown, isMediaBreakpointUp } from "../../../Utils/BreakpointsUtils";
    import { peerStore } from "../../../Stores/PeerStore";

    function closeCoWebsite() {
        if ($highlightedEmbedScreen?.type === "cowebsite") {
            if ($highlightedEmbedScreen.embed.isClosable()) {
                coWebsiteManager.closeCoWebsite($highlightedEmbedScreen.embed);
            } else {
                coWebsiteManager.unloadCoWebsite($highlightedEmbedScreen.embed).catch((err) => {
                    console.error("Cannot unload co-website", err);
                });
            }
        }
    }

    afterUpdate(() => {
        if ($highlightedEmbedScreen) {
            coWebsiteManager.resizeAllIframes();
        }
    });

    let layoutDom: HTMLDivElement;

    let displayCoWebsiteContainer = isMediaBreakpointDown("lg");
    let displayFullMedias = isMediaBreakpointUp("sm");

    const resizeObserver = new ResizeObserver(() => {
        displayCoWebsiteContainer = isMediaBreakpointDown("lg");
        displayFullMedias = isMediaBreakpointUp("sm");

        if (!displayCoWebsiteContainer && $highlightedEmbedScreen && $highlightedEmbedScreen.type === "cowebsite") {
            highlightedEmbedScreen.removeHighlight();
        }

        if (displayFullMedias) {
            highlightedEmbedScreen.removeHighlight();
        }
    });

    onMount(() => {
        resizeObserver.observe(layoutDom);
    });
</script>

<div id="presentation-layout" bind:this={layoutDom} class:full-medias={displayFullMedias}>
    {#if displayFullMedias}
        <div id="full-medias">
            <CamerasContainer full={true} highlightedEmbedScreen={$highlightedEmbedScreen} />
        </div>
    {:else}
        <div id="embed-left-block" class:full={$peerStore.size === 0}>
            <div id="main-embed-screen">
                {#if $highlightedEmbedScreen}
                    {#if $highlightedEmbedScreen.type === "streamable"}
                        {#key $highlightedEmbedScreen.embed.uniqueId}
                            <MediaBox
                                isHightlighted={true}
                                isClickable={true}
                                streamable={$highlightedEmbedScreen.embed}
                            />
                        {/key}
                    {:else if $highlightedEmbedScreen.type === "cowebsite"}
                        {#key $highlightedEmbedScreen.embed.getId()}
                            <div class="highlighted-cowebsite-container nes-container is-rounded screen-blocker">
                                <div
                                    id={"cowebsite-slot-" + $highlightedEmbedScreen.embed.getId()}
                                    class="highlighted-cowebsite"
                                />
                                <div class="actions">
                                    <button type="button" class="nes-btn is-error close" on:click={closeCoWebsite}
                                        >&times;</button
                                    >
                                </div>
                            </div>
                        {/key}
                    {/if}
                {/if}
            </div>
        </div>

        {#if $peerStore.size > 0}
            <CamerasContainer highlightedEmbedScreen={$highlightedEmbedScreen} />
        {/if}
    {/if}
</div>

<style lang="scss">
    #presentation-layout {
        height: 100%;
        width: 100%;
        display: flex;

        &.full-medias {
            overflow-y: auto;
            overflow-x: hidden;
        }

        #full-medias {
            overflow: hidden;
        }
    }

    #embed-left-block {
        display: flex;
        flex-direction: column;
        flex: 0 0 75%;
        height: 100%;
        width: 75%;

        &.full {
            flex: 0 0 98% !important;
            width: 98% !important;
        }
    }

    #main-embed-screen {
        height: 100%;
        margin-bottom: 3%;

        .highlighted-cowebsite {
            height: 100% !important;
            width: 100% !important;
            position: relative;
            z-index: 150;

            &-container {
                height: 100% !important;
                width: 96%;
                background-color: rgba(#000000, 0.6);
                margin: 0 !important;
                padding: 0 !important;
                .actions {
                    z-index: 151;
                    position: absolute;
                    width: 100%;
                    top: 0;
                    display: flex;
                    flex-direction: row;
                    justify-content: end;
                    gap: 2%;

                    button {
                        pointer-events: all;
                    }
                }
            }
        }
    }
</style>
