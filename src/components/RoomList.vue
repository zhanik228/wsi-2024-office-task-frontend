<template>
    <div v-for="body in sidebar_body" :key="body.name">
        <h2 :class="`sidebar__body-title`">{{ body.name }} <span class="count">{{ `${body.unavailable.length} / ${body.limit}` }}</span></h2>
        <div class="sidebar__body-names">
                <p v-for="user in body.unavailable" class="sidebar__body-name">{{ user.user.username }}</p>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        sidebar_body: {
            type: Array,
        }
    },
    watch: {
        sidebar_body() {
            console.log(this.sidebar_body)
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
    padding: 2px;
}

.count {
    border-radius: 15px;
    padding-inline: 3px;
    border: 1px solid green;
}

.sidebar__body-names {
    margin-left: 35px;
}
</style>