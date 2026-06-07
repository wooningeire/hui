<script>
import Separator from "./Separator.svelte";

</script>

<nav>
    <h1>
        <a href="/">vaie&#x2019;s headless ui components</a>
    </h1>

    <Separator />

    <a href="/Draggable">Draggable</a>
    <a href="/EntrySlider">EntrySlider</a>
    <a href="/Hotkey">Hotkey</a>
    <a href="/TextInput">TextInput</a>
</nav>

<style lang="scss">
nav {
    flex-basis: 20em;

    display: flex;
    flex-direction: column;
    align-items: stretch;
    gap: 1em;

    padding: 2em;
}
</style>