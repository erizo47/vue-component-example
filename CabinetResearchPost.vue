<template>
    <div class="research_post" :class="checked ? 'checked' : ''">
        <div class="research__post__checkbox">
            <input type="checkbox" :id='post.id' class="post__checkbox" :value="post" @change="select"
                :checked="checked" v-model="mainStore.selectedPosts">
            <label :for="post.id"></label>
        </div>
        <div class="research_post__item">{{ post.id }}</div>
        <div class="research_post__item research_post__name">{{ post.name }}</div>
        <div class="research_post__item">{{ post.types }}</div>
        <div class="research_post__item">{{ fileSize }}</div>
        <div class="research_post__item research_post__date">
            <p class="post__date__header">{{ dayAgo }} days ago</p>
            <p class="post__date__text">{{ postDateView }}</p>
        </div>
        <div class="research_post__item">
            <p class="status" :class="statusStyle">{{ status }}</p>
        </div>
        <div v-if="post.comment.length > 0" class="research_post__item post__comment"
            :class="{ 'comment_error': commentError }">{{ post.comment }}</div>
        <div v-else class="research_post__item post__comment">No comment</div>
        <div class="research_post__item">
            <div class="post_btn open">
                <router-link to="/research_view" @click="mainStore.setActiveUid(post.id)"><researchOpen /></router-link>
            </div>
            <div class="post_btn delete">
                <researchDelete />
            </div>
        </div>
    </div>
    <div class="post__btns">

    </div>
</template>

<script setup>
import researchOpen from '../components/icons/research_open.vue'
import researchDelete from '../components/icons/research_delete.vue'

import { useMainStore } from '../stores/mainStore.js'
import { ref, computed, watch } from 'vue';
const mainStore = useMainStore()

// конвертация из байтов в читабельный размер
const fileSize = computed(() => {
    return mainStore.convFilesize(props.post.uploadfilesize)
})

//проверка не пришел ли из пропсов checked, если нет установить по нажатию чекбокса
const isChecked = computed(() => props.isSelected)
watch(isChecked, () => checked.value = isChecked.value)
const checked = ref(false)
const select = () => {
    checked.value = !checked.value
    if (mainStore.isAllSelected) {
        mainStore.setAllSelected(false)
    }
}

//проверка статуса поста для изменени стилей и слота Status
const postStatus = computed(() => props.post.status)

const status = computed(() => {
    if (postStatus.value > 0) {
        return 'Processed'
    } else if (postStatus.value == 0) {
        return 'In process'
    } else if (postStatus.value < 0) {
        return 'Error'
    }
})

const statusStyle = computed(() => ({
    'processed': postStatus.value > 0,
    'in-process': postStatus.value == 0,
    'error': postStatus.value < 0
}))

//проверка статуса поста для изменени стилей Comment
const commentError = computed(() => {
    if (postStatus.value < 0) {
        return true
    }
})

//Отображение "days ago" и дата создания поста
const dateNow = Date.now();
const postDate = computed(() => parseInt(props.post.uploaded_at))

const postDateView = computed(() => {
    let a = new Date(postDate.value)
    const year = a.getFullYear();
    const month = a.getMonth();
    const date = a.getDate();
    const hour = a.getHours();
    const min = a.getMinutes() < 10 ? '0' + a.getMinutes() : a.getMinutes();
    const sec = a.getSeconds() < 10 ? '0' + a.getSeconds() : a.getSeconds();
    const time = date + ' ' + month + ' ' + year + ' ' + hour + ':' + min + ':' + sec;
    return time;
})

const dayAgo = computed(() => {
    let timeDiff = dateNow - postDate.value
    let diffDays = Math.ceil(timeDiff / (1000 * 3600 * 24));
    return diffDays
})

//объявление пропсов из cabinet.vue
const props = defineProps({
    post: {
        type: Object,
        required: true,
        default: () => { }
    },
    isSelected: {
        type: Boolean,
        required: true,
        default: false
    },

})

</script>

<style lang="scss" >
.checked {
    background: rgba(0, 137, 129, 0.1)
}

.research_post {
    border-bottom: 1px solid $Gray-Lighten-20;
    box-sizing: border-box;
    padding: 14px 0;
    display: grid;
    grid-template-columns: .5fr 1fr 1fr 1fr 1fr 1fr 1fr 2fr 1fr;
    color: $Gray-Darken-30;

    .research_post__item,
    .research__post__checkbox {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .research_post__date {
        flex-direction: column;
        font-weight: 400;
        color: $Gray-Darken-40;

        .post__date__header {
            font-size: 13px;
            line-height: 16px;
            opacity: 1;
            margin-bottom: 3px;
        }

        .post__date__text {
            opacity: 0.3;
            font-size: 12px;
            line-height: 15px;
        }
    }

    .research_post__name {
        font-weight: 600;
        font-size: 13px;
        line-height: 16px;
        text-decoration-line: underline;
        color: $Gray-Darken-40;

    }

    .post__comment {
        font-weight: 400;
        font-size: 12px;
        line-height: 15px;
        color: $Gray-Darken-20;
    }
    .post_btn {
        margin: 0px 5px;
    }
    .open:hover {
        a {
            svg {
                path {
                    fill: #008981;
                }
            }
        }
        
    }
    .delete:hover {
        margin: 0px 5px;
        cursor: pointer;
        svg {
            path {
                stroke: #008981;
            }
        }
    }

}

.post__checkbox {
    display: none;
}

.post__checkbox+label {
    cursor: pointer;
    user-select: none;
    box-sizing: border-box;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
}

.post__checkbox+label:before {
    content: '';
    display: block;
    width: 18px;
    height: 18px;
    border: 1px solid $Gray-Lighten-10;
    border-radius: 6px;
}


.post__checkbox:checked+label:before {
    content: '✔';
    color: $Green-Accent;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 18px;
    height: 18px;
    background: rgba(0, 137, 129, 0.1);
    border-radius: 6px;
}

.status {
    border-radius: 35px;
    width: 79px;
    height: 21px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-weight: 600;
    font-size: 10px;
    line-height: 13px;

}

.processed {
    background-color: $Green-Background;
    color: $Green-Accent;
}

.in-process {
    background-color: $Blue-Background;
    color: $Blue-Accent;
}

.error {
    background-color: $Red-Background;
    color: $Red-Accent;
}

.comment_error {
    color: $Red-Accent;
}
</style>