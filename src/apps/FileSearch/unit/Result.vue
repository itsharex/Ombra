<template>
    <div class="Result" @scroll="handle_scroll($event)">
        <div class="table_header">
            <span class="name">名称</span>
            <span class="path">路径</span>
            <span class="time">修改时间</span>
        </div>
        <div v-for="item in search_result" class="item" @contextmenu="fun_file_item_contextmenu($event, item)"
            @dblclick="fun_dbclick(item)">
            <span class="name" :title="item.name">
                <KIFolder w="12" h="12" v-if="item.ftype == 2"></KIFolder>
                <KIDll w="12" h="12" v-else-if="item.name.endsWith('.dll')"></KIDll>
                <KIText w="12" h="12" v-else-if="item.name.endsWith('.txt')"></KIText>
                <KITypeScript w="12" h="12" style="color: #4f9aba;" v-else-if="item.name.endsWith('.ts')">
                </KITypeScript>
                <KIHtml w="12" h="12" style="color: #e37933;" v-else-if="item.name.endsWith('.html')"></KIHtml>
                <span v-html="fun_show_file_name(item.name)"></span>
            </span>
            <span class="path" :title="item.path">{{ item.path }}</span>
            <span class="time">{{ time_to_str(item.time) }}</span>
        </div>
    </div>
</template>

<script setup lang="ts">
import { listen } from '@tauri-apps/api/event';
import { KIDll, KIText, KIFolder, KITypeScript, KIHtml } from '~/kui'
import { reactive } from 'vue';
import { get_span, time_to_str } from '~/global';
import { exp_open_file } from '~/ombra';
type FileInfo = {
    name: string,
    path: string,
    time: number,
    ftype: number,
}

const props = defineProps(['last_cnt', 'last_mode']);

const emits = defineEmits(['fun_set_pop_menu', 'fun_search', 'fun_complete_search']);

const search_result = reactive([]) as Array<FileInfo>;

function clear_result() {
    search_result.length = 0;
}

defineExpose({
    clear_result
})

function fun_file_item_contextmenu(e: MouseEvent, item: FileInfo) {
    emits('fun_set_pop_menu', e.clientX, e.clientY, item);
}

let scroll_count = 0;

//获取搜索结果
listen<Array<FileInfo>>('search_file_result', (e) => {
    if (e.payload.length < 50) scroll_count = -1;// 到底部了
    search_result.push(...e.payload);
    emits('fun_complete_search');
})


function handle_scroll(e: Event) {
    if (scroll_count == -1) return;
    const { scrollTop, clientHeight, scrollHeight } = e.target as HTMLElement;
    if (Math.ceil(scrollTop) + clientHeight >= scrollHeight) {
        scroll_count++;
        emits('fun_search', props.last_cnt, props.last_mode, scroll_count * 50);
    }
}

function fun_show_file_name(name: string) {
    if (props.last_mode == 'normal') {
        let pos = name.toLowerCase().indexOf(props.last_cnt.toLowerCase());
        let s = get_span(name.substring(0, pos), 'normal');
        s += get_span(name.substring(pos, pos + props.last_cnt.length), 'light');
        s += get_span(name.substring(pos + props.last_cnt.length), 'normal');
        return s;
    } else if (props.last_mode == 'regex') {
        return get_span(name, 'normal');
    } else if (props.last_mode == 'whole_word') {
        let r = RegExp(`(.*\\b)(${props.last_cnt})(\\b.*)`);
        let m = r.exec(name);
        if (m != null) {
            let s = get_span(m[1], 'normal');
            s += get_span(m[2], 'light');
            s += get_span(m[3], 'normal');
            return s;
        }

    }
    return get_span(name, 'normal');
}
function fun_dbclick(item: FileInfo) {
    if (item.ftype == 2) { //如果是文件夹
        let path = '';
        if (item.path.length > 0) {
            path = `${item.path}\\${item.name}`;
        } else {
            path = `${item.name}`;
        }
        exp_open_file(path);
        return;
    }
    let path = `${item.path}\\${item.name}`;
    exp_open_file(path);
}
</script>

<style scoped lang="less">
.Result {
    height: 100px;
    flex-grow: 1;
    color: #aaa;
    overflow-y: auto;
    overflow-x: hidden;
    user-select: none;

    &::-webkit-scrollbar {
        width: 6px;
    }

    &::-webkit-scrollbar-thumb {
        background-color: #666;
        border-radius: 3px;
    }

    &::-webkit-scrollbar-track {
        background-color: #181818;
    }

    .table_header {
        margin-top: 15px;
        display: flex;
        font-size: 12px;
        padding: 0 10px;

        span {
            display: inline-block;
        }

        .name {
            width: 200px;
        }

        .path {
            width: 400px;
            margin: 0 10px;
        }

        .time {
            flex-grow: 1;
        }

    }

    .item {
        font-size: 12px;
        display: flex;
        padding: 0 10px;
        height: 24px;

        &:hover {
            background-color: #454545;
            cursor: pointer;
        }

        span {
            white-space: nowrap;
            display: inline-block;
            overflow: hidden;
            text-overflow: ellipsis;
        }

        .name {
            width: 200px;
            display: flex;
            align-items: center;

            .KIcon {
                margin-right: 3px;
                display: flex;
                align-items: center;
            }

            :deep(.normal) {
                color: #aaa;
            }

            :deep(.light) {
                color: #4fc1ff;
            }

        }

        .path {
            width: 400px;
            margin: 0 10px;
            line-height: 24px;
        }

        .time {
            flex-grow: 1;
            line-height: 24px;
        }
    }
}
</style>~/global