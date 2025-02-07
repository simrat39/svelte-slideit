<script>
    import Glide from "@glidejs/glide";
    import { createEventDispatcher } from "svelte";

    const dispatch = createEventDispatcher();

    export let items = [];
    export let options = {};
    export let bullet = false;
    export let control = false;

    let glide;
    let controller;
    let active;

    function initGlide(node, options) {
        const events = [
            "mount.before",
            "run",
            "mount.after",
            "update",
            "play",
            "pause",
            "build.before",
            "build.after",
            "run.before",
            "run.after",
            "run.offset",
            "run.start",
            "run.end",
            "move",
            "move.after",
            "resize",
            "swipe.start",
            "swipe.move",
            "swipe.end",
            "translate.jump",
        ];

        glide = new Glide(node, options);

        // forward glide event
        events.forEach(function (event) {
            glide.on(event, function (args) {
                glide = glide; // reactive
                // Replace event name from a.b to aB or keep source
                dispatch(
                    event.replace(/\.\w/, (v) => v[1].toUpperCase()),
                    args
                );
            });
        });

        glide.on("mount.before", function () {
            dispatch("startup", glide);
        });

        glide.mount();

        glide.on("move", function () {
            dispatch("activeChanged", glide.index);
        });

        return {
            update: glide.update,
            destroy: glide.disable,
        };
    }

    function bulletIn(index, glide) {
        return {
            focus: () => glide.go(`=${index}`),
            isActive: glide.index === index,
        };
    }
</script>

<div use:initGlide={options} class="glide">
    <div class="glide__track" data-glide-el="track">
        <ul class="glide__slides">
            {#each items as item, index}
                <li class="glide__slide">
                    <slot name="item" {item} {glide} {index}>{index}</slot>
                </li>
            {/each}
        </ul>
    </div>
    {#if control && glide}
        <div class="glide__arrows">
            <slot name="control" {glide}>
                <button
                    on:click={() => glide.go("<")}
                    class="glide__arrow glide__arrow--left">prev</button
                >
                <button
                    on:click={() => glide.go("<")}
                    class="glide__arrow glide__arrow--right">next</button
                >
            </slot>
        </div>
    {/if}
    {#if bullet && glide}
        <div class="glide__bullets">
            {#each items as item, index}
                <slot name="bullet" prop={bulletIn(index, glide)}>
                    <button
                        class:focus={bulletIn(index, glide).isActive}
                        class="glide__bullet"
                        on:click={() => bulletIn(index, glide).focus()}
                    />
                </slot>
            {/each}
        </div>
    {/if}
</div>

<!-- your code here -->
<style src="./glide.core.min.css"></style>
