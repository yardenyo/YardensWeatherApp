<template>
	<div v-if="!loading" class="city-weather">
		<div class="container">
			<div class="header">
				<div class="city-wrapper">
					<div class="weather-icon">
						<img
							v-if="cityCurrentConditions.WeatherIcon <= 9"
							:src="`https://developer.accuweather.com/sites/default/files/0${cityCurrentConditions.WeatherIcon}-s.png`"
							alt="weather-icon" />
						<img v-else :src="`https://developer.accuweather.com/sites/default/files/${cityCurrentConditions.WeatherIcon}-s.png`" alt="weather-icon" />
					</div>
					<div class="city-name">
						<h2>{{ cityInfo.LocalizedName }}</h2>
						<h3>{{ cityInfo.Country.LocalizedName }}</h3>
					</div>
				</div>
				<div class="add-to-favorites">
					<template v-if="!favoriteExists()">
						<i v-tooltip.top="'Add To Favorites'" class="pi pi-heart" @click="addToFavorites"></i>
					</template>
					<template v-else>
						<i v-tooltip.top="'Remove From Favorites'" class="pi pi-heart-fill" @click="removeFromFavorites"></i>
					</template>
				</div>
			</div>
			<div class="details-section">
				<div class="current-weather">
					<div class="current-weather-info">
						<div class="current-weather-temp">
							{{ todayForecastTemprature }}
						</div>
						<div class="current-weather-text">{{ cityCurrentConditions.WeatherText }}</div>
					</div>
				</div>
			</div>
			<div class="forecast-section">
				<ForeCast :cityForecast="cityForecast" />
			</div>
		</div>
	</div>
	<LoadingComponent v-else />
</template>

<script>
import { ref, defineComponent, computed, onMounted } from "vue";
import { useWeatherStore } from "@/stores/weather.store";
import { useTemperatureStore } from "@/stores/temperature.store";
import { useFavoritesStore } from "@/stores/favorites.store";
import { storeToRefs } from "pinia";
import helpers from "@/helpers/app.helpers";
import ForeCast from "@/components/ForeCast.vue";
import LoadingComponent from "@/components/LoadingComponent.vue";

export default defineComponent({
	name: "cityWeather",
	components: {
		ForeCast,
		LoadingComponent,
	},
	props: {
		location: {
			type: Object,
			required: true,
		},
	},
	setup(props) {
		const weatherStore = useWeatherStore();
		const temperatureStore = useTemperatureStore();
		const favoritesStore = useFavoritesStore();
		const { cityInfo, cityCurrentConditions, cityForecast, searched } = storeToRefs(weatherStore);
		const { temperature } = storeToRefs(temperatureStore);
		const loading = ref(false);

		const isLocationSet = computed(() => {
			return !helpers.isNumpty(props.location);
		});

		const todayForecastTemprature = computed(() => {
			return temperature.value === "celsius"
				? `${cityCurrentConditions.value.Temperature.Metric.Value}°${cityCurrentConditions.value.Temperature.Metric.Unit}`
				: `${cityCurrentConditions.value.Temperature.Imperial.Value}°${cityCurrentConditions.value.Temperature.Imperial.Unit}`;
		});

		function favoriteExists() {
			const answer = favoritesStore.favoriteExists(cityInfo.value.Key);
			return answer;
		}

		async function addToFavorites() {
			await favoritesStore.addToFavorites(cityInfo.value);
			helpers.sendSuccessMessage(`Added ${cityInfo.value.LocalizedName} to favorites`);
		}

		async function removeFromFavorites() {
			await favoritesStore.removeFromFavorites(cityInfo.value.Key);
			helpers.sendSuccessMessage(`Removed ${cityInfo.value.LocalizedName} from favorites`);
		}

		onMounted(async () => {
			if (searched.value) return;
			loading.value = true;
			if (!isLocationSet.value) {
				await weatherStore.getCityByQuery("telaviv");
				await weatherStore.getCurrentConditions(cityInfo.value.Key);
				await weatherStore.getCityForecast(cityInfo.value.Key);
			} else {
				await weatherStore.getWeatherConditionsByGeoPosition(props.location);
				await weatherStore.getCurrentConditions(cityInfo.value.Key);
				await weatherStore.getCityForecast(cityInfo.value.Key);
			}
			loading.value = false;
		});

		return {
			loading,
			isLocationSet,
			cityInfo,
			cityForecast,
			cityCurrentConditions,
			todayForecastTemprature,
			favoriteExists,
			addToFavorites,
			removeFromFavorites,
		};
	},
});
</script>

<style lang="scss" scoped>
$spacing-unit: 1rem;
$glass-bg: rgba(255, 255, 255, 0.1);
$glass-backdrop: blur(10px);

.city-weather {
	display: flex;
	flex-direction: column;
	gap: $spacing-unit;
	padding: $spacing-unit;
	box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
	width: 100%;
	max-width: 1600px;
	margin: 0 auto;
	background: $glass-bg;
	backdrop-filter: $glass-backdrop;
	border-radius: 0.5rem;
	border: 1px solid rgba(255, 255, 255, 0.2);

	.container {
		display: flex;
		flex-direction: column;
		gap: $spacing-unit;

		.header {
			display: flex;
			align-items: center;
			justify-content: space-between;
			gap: $spacing-unit;
			padding: $spacing-unit;
			background: $glass-bg;
			backdrop-filter: $glass-backdrop;
			border-radius: 0.5rem;
			border: 1px solid rgba(255, 255, 255, 0.2);

			.city-wrapper {
				display: flex;
				align-items: center;
				gap: $spacing-unit;
			}

			.add-to-favorites {
				.pi-heart {
					cursor: pointer;
					font-size: 1.5rem;

					&:hover {
						color: $danger-primary;
					}
				}

				.pi-heart-fill {
					cursor: pointer;
					font-size: 1.5rem;
					color: $danger-primary;
				}
			}
		}

		.details-section {
			display: flex;
			flex-direction: column;
			gap: $spacing-unit;
			padding: $spacing-unit;
			background: $glass-bg;
			backdrop-filter: $glass-backdrop;
			border-radius: 0.5rem;
			border: 1px solid rgba(255, 255, 255, 0.2);

			.current-weather {
				display: flex;
				align-items: center;
				justify-content: center;
				gap: $spacing-unit;

				.current-weather-info {
					display: flex;
					flex-direction: column;
					align-items: center;
					gap: $spacing-unit;

					.current-weather-temp {
						font-size: 2rem;
						font-weight: 700;
					}

					.current-weather-text {
						font-size: 1.5rem;
						font-weight: 500;
					}
				}
			}
		}
	}
}
</style>
