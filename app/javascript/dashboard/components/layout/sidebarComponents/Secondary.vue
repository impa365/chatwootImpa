<script>
import { frontendURL } from '../../../helper/URLHelper';
import SecondaryNavItem from './SecondaryNavItem.vue';
import AccountContext from './AccountContext.vue';
import { mapGetters } from 'vuex';
import { FEATURE_FLAGS } from '../../../featureFlags';
import {
  getUserPermissions,
  hasPermissions,
} from '../../../helper/permissionsHelper';
import { routesWithPermissions } from '../../../routes';
import Policy from '../../policy.vue';

export default {
  components: {
    AccountContext,
    SecondaryNavItem,
    Policy,
  },
  props: {
    accountId: {
      type: Number,
      default: 0,
    },
    labels: {
      type: Array,
      default: () => [],
    },
    inboxes: {
      type: Array,
      default: () => [],
    },
    teams: {
      type: Array,
      default: () => [],
    },
    customViews: {
      type: Array,
      default: () => [],
    },
    menuConfig: {
      type: Object,
      default: () => {},
    },
    currentUser: {
      type: Object,
      default: () => {},
    },
    isOnChatwootCloud: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    ...mapGetters({
      isFeatureEnabledonAccount: 'accounts/isFeatureEnabledonAccount',
      currentRole: 'getCurrentRole',
      dashboardApps: 'dashboardApps/getRecords',
    }),
    hasSecondaryMenu() {
      return this.menuConfig.menuItems && this.menuConfig.menuItems.length;
    },
    contactCustomViews() {
      return this.customViews.filter(view => view.filter_type === 'contact');
    },
    accessibleMenuItems() {
      if (!this.currentRole) {
        return [];
      }
      const menuItemsFilteredByPermissions = this.menuConfig.menuItems.filter(
        menuItem => {
          const userPermissions = getUserPermissions(
            this.currentUser,
            this.accountId
          );
          return hasPermissions(
            routesWithPermissions[menuItem.toStateName],
            userPermissions
          );
        }
      );
      return menuItemsFilteredByPermissions.filter(item => {
        if (item.showOnlyOnCloud) {
          return this.isOnChatwootCloud;
        }
        return true;
      });
    },

    hideAllInboxForAgents() {
      return (
        this.isFeatureEnabledonAccount(
          this.accountId,
          'hide_all_inbox_for_agent'
        ) && this.currentRole !== 'administrator'
      );
    },
    inboxSection() {
      if (this.hideAllInboxForAgents && this.currentRole !== 'administrator') {
        return {};
      }
      return {
        icon: 'folder',
        label: 'INBOXES',
        hasSubMenu: true,
        newLink: this.showNewLink(FEATURE_FLAGS.INBOX_MANAGEMENT),
        newLinkTag: 'NEW_INBOX',
        key: 'inbox',
        toState: frontendURL(`accounts/${this.accountId}/settings/inboxes/new`),
        toStateName: 'settings_inbox_new',
        newLinkRouteName: 'settings_inbox_new',
        children: this.inboxes
          .map(inbox => ({
            id: inbox.id,
            label: inbox.name,
            truncateLabel: true,
            toState: frontendURL(
              `accounts/${this.accountId}/inbox/${inbox.id}`
            ),
            type: inbox.channel_type,
            phoneNumber: inbox.phone_number,
            reauthorizationRequired: inbox.reauthorization_required,
          }))
          .sort((a, b) =>
            a.label.toLowerCase() > b.label.toLowerCase() ? 1 : -1
          ),
      };
    },
    labelSection() {
      const userPermissions = getUserPermissions(
        this.currentUser,
        this.accountId
      );
      const canUseLabels = hasPermissions(['label_usage'], userPermissions);
      const canManageLabels = hasPermissions(['label_manage'], userPermissions);
      const isAdmin = this.currentRole === 'administrator';

      if (!canUseLabels && !isAdmin && !canManageLabels) {
        return null;
      }

      return {
        icon: 'number-symbol',
        label: 'LABELS',
        hasSubMenu: true,
        newLink: this.showNewLink(FEATURE_FLAGS.TEAM_MANAGEMENT),
        newLinkTag: 'NEW_LABEL',
        key: 'label',
        toState: frontendURL(`accounts/${this.accountId}/settings/labels`),
        toStateName: 'labels_list',
        showModalForNewItem: true,
        modalName: 'AddLabel',
        dataTestid: 'sidebar-new-label-button',
        children: this.labels.map(label => ({
          id: label.id,
          label: label.title,
          color: label.color,
          truncateLabel: true,
          toState: frontendURL(
            `accounts/${this.accountId}/label/${label.title}`
          ),
        })),
      };
    },
    dashboardAppsSection() {
      return {
        icon: 'folder',
        label: 'APPS',
        hasSubMenu: true,
        newLink: this.showNewLink(FEATURE_FLAGS.TEAM_MANAGEMENT),
        newLinkTag: 'NEW_APP',
        key: 'sidedashapp',
        // type: inbox.channel_type,
        toState: frontendURL(
          `accounts/${this.accountId}/settings/integrations/dashboard_apps`
        ),
        toStateName: 'apps_list',
        showModalForNewItem: false,
        modalName: 'DashboardAppModal',
        children: this.dashboardApps
          .filter(d => d.content[0].sidebar === true)
          .map(d => ({
            id: d.id,
            label: d.title,
            url: d.content[0].url,
            truncateLabel: true,
            toState: frontendURL(`accounts/${this.accountId}/dashApp/${d.id}`),
          })),
      };
    },
    contactLabelSection() {
      const userPermissions = getUserPermissions(
        this.currentUser,
        this.accountId
      );
      const canUseLabels = hasPermissions(['label_usage'], userPermissions);
      const canManageLabels = hasPermissions(['label_manage'], userPermissions);
      const isAdmin = this.currentRole === 'administrator';

      if (!canUseLabels && !isAdmin && !canManageLabels) {
        return null;
      }

      return {
        icon: 'number-symbol',
        label: 'TAGGED_WITH',
        hasSubMenu: true,
        key: 'label',
        newLink: this.showNewLink(FEATURE_FLAGS.TEAM_MANAGEMENT),
        newLinkTag: 'NEW_LABEL',
        toState: frontendURL(`accounts/${this.accountId}/settings/labels`),
        toStateName: 'labels_list',
        showModalForNewItem: true,
        modalName: 'AddLabel',
        children: this.labels.map(label => ({
          id: label.id,
          label: label.title,
          color: label.color,
          truncateLabel: true,
          toState: frontendURL(
            `accounts/${this.accountId}/labels/${label.title}/contacts`
          ),
        })),
      };
    },
    teamSection() {
      return {
        icon: 'people-team',
        label: 'TEAMS',
        hasSubMenu: true,
        newLink: this.showNewLink(FEATURE_FLAGS.TEAM_MANAGEMENT),
        newLinkTag: 'NEW_TEAM',
        key: 'team',
        toState: frontendURL(`accounts/${this.accountId}/settings/teams/new`),
        toStateName: 'settings_teams_new',
        newLinkRouteName: 'settings_teams_new',
        children: this.teams.map(team => ({
          id: team.id,
          label: team.name,
          truncateLabel: true,
          toState: frontendURL(`accounts/${this.accountId}/team/${team.id}`),
        })),
      };
    },
    foldersSection() {
      return {
        icon: 'folder',
        label: 'CUSTOM_VIEWS_FOLDER',
        hasSubMenu: true,
        key: 'custom_view',
        children: this.customViews
          .filter(view => view.filter_type === 'conversation')
          .map(view => ({
            id: view.id,
            label: view.name,
            truncateLabel: true,
            toState: frontendURL(
              `accounts/${this.accountId}/custom_view/${view.id}`
            ),
          })),
      };
    },
    contactSegmentsSection() {
      return {
        icon: 'folder',
        label: 'CUSTOM_VIEWS_SEGMENTS',
        hasSubMenu: true,
        key: 'custom_view',
        children: this.customViews
          .filter(view => view.filter_type === 'contact')
          .map(view => ({
            id: view.id,
            label: view.name,
            truncateLabel: true,
            toState: frontendURL(
              `accounts/${this.accountId}/contacts/custom_view/${view.id}`
            ),
          })),
      };
    },
    additionalSecondaryMenuItems() {
      let conversationMenuItems = [this.inboxSection, this.labelSection].filter(
        item => item
      );
      let contactMenuItems = [this.contactLabelSection].filter(item => item);

      if (this.teams.length) {
        conversationMenuItems = [
          this.teamSection,
          ...conversationMenuItems,
        ].filter(item => item);
      }
      if (this.customViews.length) {
        conversationMenuItems = [this.foldersSection, ...conversationMenuItems];
      }
      if (
        this.dashboardApps.filter(d => d.content[0].sidebar === true).length
      ) {
        conversationMenuItems = [
          this.dashboardAppsSection,
          ...conversationMenuItems,
        ].filter(item => item);
      }
      if (this.contactCustomViews.length) {
        contactMenuItems = [
          this.contactSegmentsSection,
          ...contactMenuItems,
        ].filter(item => item);
      }
      return {
        conversations: conversationMenuItems,
        contacts: contactMenuItems,
      };
    },
  },
  mounted() {
    this.$store.dispatch('dashboardApps/get');
  },
  methods: {
    showAddLabelPopup() {
      this.$emit('addLabel');
    },
    showDashboardAppPopup() {
      this.$emit('DashboardAppModal');
    },
    toggleAccountModal() {
      this.$emit('toggleAccounts');
    },
    showNewLink(featureFlag) {
      return this.isFeatureEnabledonAccount(this.accountId, featureFlag);
    },
    getPolicyPermissions(newLinkTag) {
      if (newLinkTag === 'NEW_LABEL') {
        return ['administrator', 'label_manage'];
      }
      return ['administrator'];
    },
    newLinkClick(event, navigate) {
      event.preventDefault();
      navigate();
    },
  },
};
</script>

<template>
  <div
    v-if="hasSecondaryMenu"
    class="flex flex-col w-48 h-full px-2 pb-8 overflow-auto text-sm bg-white border-r dark:bg-slate-900 dark:border-slate-800/50 rtl:border-r-0 rtl:border-l border-slate-50"
  >
    <AccountContext @toggleAccounts="toggleAccountModal" />
    <transition-group
      name="menu-list"
      tag="ul"
      class="pt-2 mb-0 ml-0 list-none"
    >
      <SecondaryNavItem
        v-for="menuItem in accessibleMenuItems"
        :key="menuItem.toState"
        :menu-item="menuItem"
      />
      <SecondaryNavItem
        v-for="menuItem in additionalSecondaryMenuItems[menuConfig.parentNav]"
        :key="menuItem.key"
        :menu-item="menuItem"
        @addLabel="showAddLabelPopup"
      >
        <Policy :permissions="getPolicyPermissions(menuItem.newLinkTag)">
          <router-link
            v-if="menuItem.newLink"
            v-slot="{ href, navigate }"
            :to="menuItem.toState"
            custom
          >
            <li class="pl-1">
              <a :href="href">
                <woot-button
                  size="tiny"
                  variant="clear"
                  color-scheme="secondary"
                  icon="add"
                  :data-testid="menuItem.dataTestid"
                  @click="e => newLinkClick(e, navigate)"
                >
                  {{ $t(`SIDEBAR.${menuItem.newLinkTag}`) }}
                </woot-button>
              </a>
            </li>
          </router-link>
        </Policy>
      </SecondaryNavItem>
    </transition-group>
  </div>
</template>
