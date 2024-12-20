<template>
	<div class="w-full mt-2">
		<div class="flex flex-col gap-2">
			<div class="grid grid-cols-1 sm:grid-cols-3 gap-2 sm:gap-4 items-end">
				<div
					v-for="(pago, index) in evento.precios"
					:key="index"
					class="flex flex-row sm:flex-col"
				>
					<label
						:for="`cantidad-personas${index}`"
						class="text-sm w-full sm:w-3/4"
					>
						{{ pago.tipo }}
					</label>
					<InputNumber
						inputClass="h-8 w-full"
						:inputId="`cantidad-personas${index}`"
						:invalid="invalid"
						v-model="reserva.cantidad[index].cantidad"
						showButtons
						buttonLayout="horizontal"
						:min="0"
						:max="
							cuposRestantes <= 0
								? reserva.cantidad[index].cantidad
								: horario.spots
						"
					>
						<template #incrementbuttonicon>
							<span class="pi pi-plus" />
						</template>
						<template #decrementbuttonicon>
							<span class="pi pi-minus" />
						</template>
					</InputNumber>
				</div>
			</div>
			<Fieldset
				legend="Adicionales"
				:toggleable="true"
				:collapsed="true"
				v-on:toggle="console.log($event.value ? 'collapsed' : 'expanded')"
			>
				<div class="grid grid-cols-1 sm:grid-cols-3 gap-4 items-end">
					<div
						class="flex flex-row sm:flex-col mb-2"
						v-for="(pago, index) in evento.precios"
						:key="index"
					>
						<label
							:for="`cantidad-adicionales${index}`"
							class="text-sm mt-2 w-full sm:w-3/4"
						>
							{{ pago.tipo }}
						</label>
						<InputNumber
							inputClass="h-8 w-full"
							:inputId="`cantidad-adicionales${index}`"
							:invalid="invalid"
							v-model="reserva.cantidadAdicional[index].cantidad"
							showButtons
							buttonLayout="horizontal"
							:min="0"
						>
							<template #incrementbuttonicon>
								<span class="pi pi-plus" />
							</template>
							<template #decrementbuttonicon>
								<span class="pi pi-minus" />
							</template>
						</InputNumber>
					</div>
				</div>
			</Fieldset>
		</div>
		<Divider type="dashed" />
		<div class="flex flex-col gap-2">
			<div class="card flex flex-col">
				<label for="tipoReserva" class="text-sm">Tipo de reserva</label>
				<SelectButton
					id="tipoReserva"
					v-model="reserva.tipoReserva"
					:options="tiposDeReserva"
					optionLabel="label"
					optionValue="value"
				/>
			</div>
			<div class="grid grid-cols-1 sm:grid-cols-2 gap-2 sm:gap-4">
				<div class="flex flex-col">
					<label for="paymethod" class="text-sm">Método de pago</label>
					<Select
						id="paymethod"
						v-model="reserva.pago.metodoPago"
						:options="paymethod"
						optionLabel="name"
						optionValue="value"
						placeholder="Seleccione un metodo de pago"
						class="h-8 w-full"
						:disabled="reserva.tipoReserva === 'call'"
					/>
				</div>
				<div class="flex flex-col">
					<label for="descuento" class="text-sm">Descuento</label>
					<InputNumber
						inputClass="h-8 w-full"
						v-model="reserva.pago.descuento"
						inputId="descuento"
						mode="decimal"
						showButtons
						fluid
						:min="0"
					/>
				</div>
			</div>
		</div>
		<Divider type="dashed" />
		<div class="flex flex-col gap-2">
			<div>
				<label for="nombre-cliente" class="text-sm">Nombre completo</label>
				<InputText
					class="uppercase h-8"
					type="text"
					id="nombre-cliente"
					fluid
					v-model="reserva.cliente.nombre"
					:invalid="reserva.cliente.nombre === '' && invalid"
				/>
			</div>
			<div>
				<label for="correo-cliente" class="text-sm">Correo electrónico</label>
				<InputText
					class="lowercase h-8"
					type="email"
					id="correo-cliente"
					fluid
					v-model="reserva.cliente.email"
					:invalid="reserva.cliente.email === '' && invalid"
				/>
			</div>
			<div>
				<label for="lugar-cliente" class="text-sm"
					>Desde donde nos visitas</label
				>
				<InputGroup class="h-8">
					<Select
						v-model="selectedPlace.pais"
						:options="countries"
						optionLabel="country"
						placeholder="🌎"
						class="w-1/3"
						labelClass="text-sm pt-[0.4rem]"
						v-on:update:model-value="loadStates($event.code)"
					>
					</Select>
					<Select
						v-model="selectedPlace.estado"
						:options="world.states"
						optionLabel="name"
						placeholder="Seleccione..."
						class="w-1/3"
						labelClass="text-sm pt-[0.4rem]"
						v-on:update:model-value="loadCities($event.id)"
						:disabled="world.states.length === 0"
						:loading="world.states.length === 0 && interactividad.loading"
					>
					</Select>
					<Select
						v-model="reserva.cliente.ciudad"
						:options="world.cities"
						optionLabel="name"
						optionValue="name"
						placeholder="Seleccione..."
						class="w-1/3"
						labelClass="text-sm pt-[0.4rem]"
						:disabled="world.cities.length === 0"
						:loading="world.cities.length === 0 && interactividad.loading"
					>
					</Select>
				</InputGroup>
			</div>
			<div>
				<label for="numero-cliente" class="text-sm">Número de teléfono</label>
				<InputGroup class="h-8">
					<Select
						v-model="selectedCountry"
						:options="countries"
						optionLabel="country"
						placeholder="🌎"
						class="w-2/5 sm:w-3/12"
						labelClass="text-sm pt-[0.4rem]"
					>
						<template #value="slotProps">
							<div v-if="slotProps.value" class="flex items-center">
								<img
									:alt="slotProps.value.country"
									:src="slotProps.value.flag"
									:class="`mr-2 flag flag-${slotProps.value.code.toLowerCase()}`"
									style="width: 18px"
								/>
								<div>
									{{ `(+${slotProps.value.countryCode})` }}
								</div>
							</div>
							<span v-else>
								{{ slotProps.placeholder }}
							</span>
						</template>
						<template #option="slotProps">
							<div class="flex items-center">
								<img
									:alt="slotProps.option.label"
									:src="slotProps.option.flag"
									:class="`mr-2`"
									style="width: 18px"
								/>
								<div>
									{{ slotProps.option.country }} (+
									{{ slotProps.option.countryCode }})
								</div>
							</div>
						</template>
					</Select>
					<InputNumber
						v-model="numero"
						:invalid="numero === null && invalid"
						inputId="numero-cliente"
						:useGrouping="false"
						class="w-3/5 sm:w-9/12"
					/>
				</InputGroup>
			</div>
		</div>
	</div>
</template>
<script setup>
import { ref, computed, watch } from 'vue'
import { useEventos } from '@/composables/useEventos'
import { useReservas } from '@/composables/useReservas'
import countries from '@/assets/countries.json'

const { evento } = useEventos()
const {
	reserva,
	horario,
	world,
	selectedPlace,
	interactividad,
	loadCities,
	loadStates,
} = useReservas()

const invalid = ref(false)
const selectedCountry = ref({
	country: 'Bolivia',
	countryCode: 591,
	code: 'BO',
	flag: 'https://flagcdn.com/bo.svg',
})
const numero = ref(null)
const paymethod = ref([
	{ name: 'Efectivo', value: 'efectivo' },
	{ name: 'QR', value: 'qr' },
])
const tiposDeReserva = ref([
	{ label: '🏢 En persona', value: 'onsite' },
	{ label: '📲 Por Llamada/WhatsApp', value: 'call' },
])

const cuposRestantes = computed(() => {
	return (
		horario.value.spots -
		reserva.value.cantidad.reduce((acc, curr) => acc + curr.cantidad, 0)
	)
})

watch(selectedPlace.value, (value) => {
	const { pais, estado } = value
	reserva.value.cliente.pais = pais.country
	reserva.value.cliente.estado = estado.name
})

watch(
	() => `${selectedCountry.value.countryCode}${numero.value}`,
	(value) => {
		reserva.value.cliente.telefono = value
	}
)
</script>
