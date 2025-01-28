<script>
export default {
    methods: {
        close() {
            this.$emit("close");
        },
    },
};
</script>

<template>
    <div class="modal-overlay" @click.self="close">
        <div class="modal">
            <header>
                <slot name="header"></slot>
                <button @click="close">X</button>
            </header>
            <section>
                <slot name="body"></slot>
            </section>
            <footer>
                <slot name="footer"></slot>
            </footer>
        </div>
    </div>
</template>

<style scoped>
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 10;
    display: flex;
    justify-content: center;
    align-items: center;
}

.modal {
    width: 90%;
    max-width: 600px;
    max-height: 80vh;
    overflow-y: auto;
    background-color: white;
    border-radius: 8px;
    padding: 20px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
    position: relative;
}

header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid #ddd;
    padding-bottom: 10px;
    margin-bottom: 10px;
}

header h3 {
    margin: 0;
    font-size: 1.25em;
}

button {
    background-color: transparent;
    border: none;
    cursor: pointer;
    font-size: 1.25em;
}

footer {
    display: flex;
    justify-content: flex-end;
    padding-top: 10px;
    border-top: 1px solid #ddd;
    margin-top: 10px;
}
</style>
