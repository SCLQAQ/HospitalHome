<template>
    <Layout class="main-layer">
        <Sider ref="side1" hide-trigger collapsible :collapsed-width="78" v-model="isCollapsed">
            <Affix>
                <div class="logo"></div>
                <Menu ref="menu" theme="dark" width="auto" :class="menuitemClasses" :open-names="openMenus.map(e => e.ActionName || e.Text)" :active-name="activeMenu">
                    <template v-for="(item, index) in menus">
                        <i-menu-item v-if="item.Items && !item.Items.length" :key="index" :to="{ name: item.ActionName }" :name="item.ActionName || item.Text">
                            <i :class="{fa: true, [item.Icons]: true}" :key="'i' + index"/>
                            <span :key="'span' + index">{{ item.Text }}</span>
                        </i-menu-item>
                        <Submenu v-if="item.Items && item.Items.length" :key="index" :name="item.Text">
                            <template slot="title">
                                <i :class="{fa: true, [item.Icons]: true}"/>
                                <span>{{ item.Text }}</span>
                            </template>
                            <i-menu-item v-for="(v, k) in item.Items" :key="k" :to="{ name: v.ActionName }" :name="v.ActionName || v.Text">
                                <!-- <i :class="{fa: true, [v.Icons]: true}"/> -->
                                <span>{{ v.Text }}</span>
                            </i-menu-item>
                        </Submenu>
                    </template>
                </Menu>
            </Affix>
        </Sider>
        <Layout>
            <Affix>
                <Header :style="{padding: 0}" class="layout-header-bar">
                    <Icon @click.native="collapsedSider" :class="rotateIcon" :style="{margin: '0 20px'}" type="md-menu" size="24"></Icon>
                </Header>
            </Affix>
            <Content :style="{margin: '20px'}" class="content">
                <slot></slot>
            </Content>
            <Footer class="layout-footer-center">&copy; 2019 <a href="http://www.ricebird.cn">厦门米雀软件科技有限公司</a> 版权所有</Footer>
        </Layout>
    </Layout>
</template>

<script>
import { Layout, Sider, Menu, MenuItem, Header, Icon, Content, Affix, Submenu } from 'iview'
const app = require('@/config')
export default {
    components: { Layout, Sider, Menu, MenuItem, Header, Icon, Content, Affix, Submenu },
    created () {
        let signalR = require("@/api/signalR").default;
        signalR.ready(msg => {
            signalR.resetUserId();
        })
    },
    mounted () {
        if (!app.checkPermission("Security.LoginAdmin")) {
            this.$Message.warning("你没有权限登录后台");
            this.$router.push({ name: "Login" });
        }
        app.getMenus(menus => {
            this.$set(app, "menus", menus);
            this.menus = app.menus;
            this.collapseMenu(this.$route.name)
        })
    },
    beforeRouteEnter (to, from, next) {
        next(vm => {
            // console.log("enter", to, from)
            vm.collapseMenu(to.name)
        });
    },
    beforeRouteUpdate (to, from, next) {
        // console.log("update", to, from)
        this.collapseMenu(to.name)
        next();
    },
    methods: {
        collapseMenu (name) {
            let menuContext = this.$refs.menu;
            for (let m of app.menus) {
                let menus = [];
                // 1. 没有子菜单的情况
                if (m.Items && !m.Items.length) {
                    if (m.ActionName === name) {
                        this.openMenus = menus;
                        this.activeMenu = name;
                        return;
                    } else {
                        continue;
                    }
                } else {
                    // 有子菜单的情况
                    menus.push(m)
                    for (let cm of m.Items) {
                        if (cm.ActionName === name) {
                            this.openMenus = menus;
                            this.activeMenu = name;
                            this.$nextTick(() => {
                                menuContext.updateOpened();
                                menuContext.updateActiveName();
                            })
                            return;
                        }
                    }
                }
            }
        },
        collapsedSider () {
            this.$refs.side1.toggleCollapse();
        }
    },
    data () {
        if (!app.toMenu) app.toMenu = this.collapseMenu;
        return {
            isCollapsed: false,
            menus: app.menus,
            openMenus: [],
            activeMenu: ""
        }
    },
    watch: {
        $route (v) {
            this.collapseMenu(this.$route.name)
        }
    },
    computed: {
        rotateIcon () {
            return [
                'menu-icon',
                this.isCollapsed ? 'rotate-icon' : ''
            ];
        },
        menuitemClasses () {
            return [
                'menu-item',
                this.isCollapsed ? 'collapsed-menu' : ''
            ]
        }
    }
}
</script>

<style scoped lang="less">
.logo {
    height: 64px;
    widows: 100%;
}
.layout-footer-center{
    text-align: center;
}

.layout{
    border: 1px solid #d7dde4;
    background: #f5f7f9;
    position: relative;
    border-radius: 4px;
    overflow: hidden;
}
.layout-header-bar{
    background: #fff;
    box-shadow: 0 1px 1px rgba(0,0,0,.1);
}
.layout-logo-left{
    width: 90%;
    height: 30px;
    background: #5b6270;
    border-radius: 3px;
    margin: 15px auto;
}
.menu-icon{
    transition: all .3s;
}
.rotate-icon{
    transform: rotate(-90deg);
}
.menu-item span{
    display: inline-block;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    vertical-align: bottom;
    transition: width .2s ease .2s;
}
.menu-item i{
    transform: translateX(0px);
    transition: font-size .2s ease, transform .2s ease;
    vertical-align: middle;
    font-size: 16px;
}
.collapsed-menu span{
    width: 0px;
    transition: width .2s ease;
}
.collapsed-menu i{
    transform: translateX(5px);
    transition: font-size .2s ease .2s, transform .2s ease .2s;
    vertical-align: middle;
    font-size: 22px;
}
.main-layer {
    min-height: fill-available;
}
</style>
