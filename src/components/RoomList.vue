<template>
    <div v-for="body in sidebar_body" :key="body.title.name">
        <h2 :class="`sidebar__body-title ${body.title.bold ? 'bold' : ''}`">{{ body.title.name }} {{ `(${body.people.length}/${body.max_amount})` }}</h2>
        <div class="sidebar__body-names">
                <p v-for="people in body.people" class="sidebar__body-name">{{ people }}</p>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        sidebar_body: {
            type: Array,
        }
    }
}
</script>

<style scoped>
.bold {
    font-weight: 700;
}

.sidebar__body-title {
    margin-left: 20px;
    font-size: 18px;
}

.sidebar__body-names {
    margin-left: 35px;
}
</style>