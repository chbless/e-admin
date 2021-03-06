<template>
  <div class="ea-header">
    <div
      v-if="collapse !== 'false'"
      class="ea-hover"
      @click="mCollapse = !mCollapse"
    >
      <i :class="`ea-icon-page-arrow-${mCollapse ? 'right' : 'left'}`"/>
    </div>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item
          v-for="route in breadcrumbs"
          :key="route.path"
          :to="route.disabled ? undefined : route.path"
        >
          {{ route.label || route.title }}
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="_center">
      <slot/>
    </div>
    <el-tooltip
      v-if="notice !== false"
      :content="noticeContent"
      effect="dark"
      placement="bottom"
    >
      <div class="ea-hover" @click="command('notice')">
        <el-badge
          :value="typeof notice !== 'boolean' ? notice : ''"
          :is-dot="notice === true"
          :max="99"
          class="_badge">
        </el-badge>
        <i class="ea-icon-notices"/>
      </div>
    </el-tooltip>
    <el-tooltip
      v-if="fullscreen"
      :content="!screenfull ? '全屏' : '还原'"
      effect="dark"
      placement="bottom">
      <div class="ea-hover" @click="fullScreen">
        <i :class="`ea-icon-fullscreen${screenfull ? '-exit' : ''}`"/>
      </div>
    </el-tooltip>
    <div class="ea-hover">
      <el-dropdown slot="reference" trigger="click" @command="command">
        <a>
          <span>{{ user.name }}</span>
          <i class="el-icon-arrow-down el-icon--right"/>
        </a>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item
            v-for="item in dropdownMenu"
            :command="item.value"
            :key="item.value"
            :divided="item.divided">
            {{ item.label }}
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
  </div>
</template>
<script>
  import screenFull from 'screenfull';
  import ConfigMixins from '../../../mixins/config';

  export default {
    name: 'EaHeader',
    mixins: [ConfigMixins({
      collapse: [Boolean, String],
      home: Object,
    })],
    props: {
      navData: {
        type: Array,
        default: () => [],
      },
      notice: {
        type: [Number, String, Boolean],
        default: '',
      },
      noticeContent: {
        type: String,
        default: '消息',
      },
      fullscreen: {
        type: Boolean,
        default: true,
      },
      user: {
        type: Object,
        default: () => ({name: '未知', avatar: ''}),
      },
      dropdownMenu: {
        type: Array,
        default: () => [
          {value: 'me', label: '我的信息', divided: false},
          {value: 'setting', label: '设置', divided: false},
          {value: 'logout', label: '退出', divided: true},
        ],
      },
    },
    data() {
      return {
        screenfull: false,
        header: true,
      };
    },
    computed: {
      breadcrumbs() {
        let to = this.$route;
        let matched = to.matched.find(v => v.meta.childrenAgent) || to.matched[to.matched.length - 1];
        let path = matched.path;
        let list = [this.mHome];
        if (path !== this.mHome.path) {
          let _list = this.getBreadcrumbByNavData(this.navData, path);
          if (_list) {
            list.push(..._list);
          } else {
            for (let i = 1; i < to.matched.length; i++) {
              let matched = to.matched[i];
              list.push({
                path: matched.path,
                title: matched.meta.tabs && matched.meta.tabs.title ? matched.meta.tabs.title : (matched.meta.label || matched.meta.title || matched.name),
              });
            }
          }
        }
        return list;
      },
    },
    methods: {
      getBreadcrumbByNavData(ary, path, list = []) {
        for (let item of ary) {
          let success = false;
          if (item.path === path) {
            list.push(item);
            success = true;
          }
          if (item.children) {
            const _list = this.getBreadcrumbByNavData(item.children, path, success ? [...list] : [...list, item]);
            if (_list) return _list;
          }
          if (success) return list;
        }
      },
      command(v) {
        this.$emit('command', v);
      },
      fullScreen() {
        if (this.screenfull) {
          this.screenfull = false;
          screenFull.exit();
        } else if (screenFull.isEnabled) {
          this.screenfull = true;
          screenFull.request();
        }
      },
    },
  };
</script>