<script setup>
import Button from 'primevue/button';
import Column from 'primevue/column';
import DataTable from 'primevue/datatable';
import InputNumber from 'primevue/inputnumber';
import InputText from 'primevue/inputtext';
import { computed, reactive, ref } from 'vue';

const IVA = 0.16;

// Productos iniciales (igual que el mockup)
const productos = ref([
    { id: 1, nombre: 'Impresora LaserJet Color', precio: 5200, cantidad: 2 },
    { id: 2, nombre: 'Monitor LED 31 plg.', precio: 1700, cantidad: 3 }
]);
let siguienteId = 3;

const form = reactive({ nombre: '', cantidad: null, precio: null });
const editandoId = ref(null); // null = registrando, número = editando
const error = ref('');

const moneda = (v) => Number(v || 0).toLocaleString('en-US', { style: 'currency', currency: 'USD' });
const redondear = (v) => Math.round((v + Number.EPSILON) * 100) / 100;

const importe = (p) => redondear(p.precio * p.cantidad);

const subtotal = computed(() => redondear(productos.value.reduce((s, p) => s + p.precio * p.cantidad, 0)));
const iva = computed(() => redondear(subtotal.value * IVA));
const total = computed(() => redondear(subtotal.value + iva.value));

function limpiar() {
    form.nombre = '';
    form.cantidad = null;
    form.precio = null;
    editandoId.value = null;
    error.value = '';
}

function guardar() {
    const nombre = (form.nombre || '').trim();
    if (!nombre || !form.cantidad || form.cantidad < 1 || !form.precio || form.precio <= 0) {
        error.value = 'Captura el nombre, una cantidad mayor a 0 y un precio mayor a 0.';
        return;
    }
    if (editandoId.value === null) {
        productos.value.push({ id: siguienteId++, nombre, cantidad: form.cantidad, precio: form.precio });
    } else {
        const p = productos.value.find((x) => x.id === editandoId.value);
        if (p) {
            p.nombre = nombre;
            p.cantidad = form.cantidad;
            p.precio = form.precio;
        }
    }
    limpiar();
}

function editar(p) {
    editandoId.value = p.id;
    form.nombre = p.nombre;
    form.cantidad = p.cantidad;
    form.precio = p.precio;
    error.value = '';
}

function eliminar(p) {
    if (!window.confirm(`¿Eliminar "${p.nombre}" de la compra?`)) return;
    productos.value = productos.value.filter((x) => x.id !== p.id);
    if (editandoId.value === p.id) limpiar();
}
</script>

<template>
    <div class="card">
        <div class="font-semibold text-xl mb-4">PUNTO DE VENTA (POS)</div>

        <!-- Formulario de registro -->
        <div class="flex flex-wrap gap-3 items-start mb-2 p-4 rounded-lg bg-surface-50 dark:bg-surface-800">
            <InputText v-model="form.nombre" placeholder="Nombre del producto..." class="flex-1 min-w-48" @keyup.enter="guardar" />
            <InputNumber v-model="form.cantidad" placeholder="Cant" :min="1" :useGrouping="false" class="w-32" @keyup.enter="guardar" />
            <InputNumber v-model="form.precio" placeholder="$ Precio U." mode="currency" currency="USD" locale="en-US" :min="0" class="w-44" @keyup.enter="guardar" />
            <Button
                :label="editandoId === null ? 'Registrar' : 'Actualizar'"
                :icon="editandoId === null ? 'pi pi-plus' : 'pi pi-check'"
                severity="success"
                @click="guardar"
            />
            <Button v-if="editandoId !== null" label="Cancelar" icon="pi pi-times" severity="secondary" @click="limpiar" />
        </div>
        <small v-if="error" class="text-red-500 block mb-3">{{ error }}</small>

        <!-- Tabla de productos -->
        <DataTable
            :value="productos"
            dataKey="id"
            paginator
            :rows="5"
            :rowsPerPageOptions="[5, 10, 20]"
            paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
            currentPageReportTemplate="Visualizando {last} de {totalRecords} productos"
            showGridlines
            class="mt-3"
        >
            <template #empty>No hay productos registrados.</template>
            <Column field="id" header="Cns" sortable style="width: 6rem" />
            <Column field="nombre" header="Nombre del Producto" sortable />
            <Column field="precio" header="Precio U." sortable>
                <template #body="{ data }">{{ moneda(data.precio) }}</template>
            </Column>
            <Column field="cantidad" header="Cant." sortable style="width: 7rem" />
            <Column header="Precio P.">
                <template #body="{ data }">{{ moneda(importe(data)) }}</template>
            </Column>
            <Column header="Acciones" style="width: 9rem">
                <template #body="{ data }">
                    <div class="flex gap-2">
                        <Button icon="pi pi-pencil" severity="success" size="small" aria-label="Editar" @click="editar(data)" />
                        <Button icon="pi pi-trash" severity="danger" size="small" aria-label="Eliminar" @click="eliminar(data)" />
                    </div>
                </template>
            </Column>
        </DataTable>

        <!-- Totales -->
        <div class="flex flex-wrap justify-end items-center gap-3 mt-4 p-4 rounded-lg bg-surface-50 dark:bg-surface-800">
            <label class="font-semibold">Subtotal:</label>
            <InputText :modelValue="moneda(subtotal)" readonly class="w-40 text-right" />
            <label class="font-semibold">IVA (16%):</label>
            <InputText :modelValue="moneda(iva)" readonly class="w-40 text-right" />
            <label class="font-semibold">Total:</label>
            <InputText :modelValue="moneda(total)" readonly class="w-40 text-right font-bold" />
        </div>
    </div>
</template>