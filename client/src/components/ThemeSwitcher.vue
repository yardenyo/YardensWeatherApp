<template>
	<div :class="isMobile ? 'theme-switcher-mobile' : 'theme-switcher'" @click="toggleTheme">
		<i v-if="!isMobile" :class="themeIcon"></i>
		<span v-else>Change Theme</span>
	</div>
</template>

<script>
import { useThemeStore } from "@/stores/theme.store";
import { storeToRefs } from "pinia";
import { computed, defineComponent } from "vue";
import helpers from "@/helpers/app.helpers";

export default defineComponent({
	name: "ThemeSwitcher",
	props: {
		isMobile: {
			type: Boolean,
			default: false,
		},
	},
	setup() {
		const themeStore = useThemeStore();
		const { theme } = storeToRefs(themeStore);

		const themeIcon = computed(() => {
			return theme.value === "light" ? "pi pi-moon" : "pi pi-sun";
		});

		async function toggleTheme() {
			await themeStore.toggleTheme();
			helpers.sendInfoMessage(`Theme changed to ${theme.value}`);
		}

		return {
			theme,
			themeIcon,
			toggleTheme,
		};
	},
});
</script>

<style lang="scss" scoped>
.theme-switcher {
	cursor: pointer;
	display: flex;
	align-items: center;
	justify-content: center;
	width: 50px;
	height: 50px;
	border-radius: 50%;
}

.theme-switcher-mobile {
	padding: 0.5rem;
	border-radius: 0.5rem;
	cursor: pointer;
	transition: all 0.2s ease-in-out;
	text-decoration: none;
}

.pi {
	font-weight: 600;
}
</style>
