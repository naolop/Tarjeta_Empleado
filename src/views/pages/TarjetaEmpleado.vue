<script setup>
import Button from 'primevue/button';
import InputText from 'primevue/inputtext';
import QrcodeVue from 'qrcode.vue';
import { reactive, ref } from 'vue';

const form = reactive({ nombre: '', paterno: '', materno: '', fecha: '', puesto: '' });
const rfc = ref('');
const foto = ref(null);
const tarjeta = ref(null);

function onFoto(e) {
    const archivo = e.target.files[0];
    if (!archivo) return;
    const lector = new FileReader();
    lector.onload = () => (foto.value = lector.result);
    lector.readAsDataURL(archivo);
}

function generarRfc() {
    const quitar = (t) => t.normalize('NFD').replace(/[\u0300-\u036f]/g, '').toUpperCase().trim();
    const p = quitar(form.paterno);
    const m = quitar(form.materno);
    const n = quitar(form.nombre);
    const vocal = (p.slice(1).match(/[AEIOU]/) || ['X'])[0];
    const [anio, mes, dia] = form.fecha.split('-');
    const letras = (p[0] || 'X') + vocal + (m[0] || 'X') + (n[0] || 'X');
    const homoclave = Math.random().toString(36).substring(2, 5).toUpperCase();
    return `${letras}${anio.slice(2)}${mes}${dia}${homoclave}`;
}

function formatearFecha(f) {
    const [a, m, d] = f.split('-');
    return `${d}/${m}/${a}`;
}

function generar() {
    if (!form.nombre || !form.paterno || !form.fecha) return;
    rfc.value = generarRfc();
    tarjeta.value = { ...form, rfc: rfc.value, fechaTxt: formatearFecha(form.fecha) };
}
</script>

<template>
    <div class="grid grid-cols-12 gap-6">
        <!-- FORMULARIO -->
        <div class="col-span-12 md:col-span-6">
            <div class="card flex flex-col gap-4">
                <div class="font-semibold text-xl">GENERAR EL RFC DE UNA PERSONA</div>
                <label>Nombre(s):</label>
                <InputText v-model="form.nombre" />
                <label>Apellido Paterno:</label>
                <InputText v-model="form.paterno" />
                <label>Apellido Materno:</label>
                <InputText v-model="form.materno" />
                <label>Fecha Nacimiento:</label>
                <InputText v-model="form.fecha" type="date" />
                <label>Puesto (opcional):</label>
                <InputText v-model="form.puesto" placeholder="Empleado" />
                <label>RFC:</label>
                <InputText :modelValue="rfc" readonly />
                <Button label="GENERAR" severity="success" @click="generar" />
            </div>
        </div>

        <!-- TARJETA -->
        <div class="col-span-12 md:col-span-6">
            <div class="card flex flex-col items-center gap-3">
                <div class="font-semibold text-xl self-start">TARJETA DIGITAL</div>

                <div v-if="tarjeta" class="id-card">
                    <!-- Encabezado -->
                    <div class="id-top">
                        <span class="circle c1"></span>
                        <span class="circle c2"></span>
                        <div class="id-brand">
                            <div class="id-logo">FN</div>
                            <div>
                                <div class="id-brand-name">FACULTAD DE NEGOCIOS</div>
                                <div class="id-brand-sub">UNACH</div>
                            </div>
                        </div>
                        <svg class="id-wave" viewBox="0 0 300 60" preserveAspectRatio="none">
                            <path d="M0,30 C80,70 180,-10 300,30 L300,60 L0,60 Z" fill="#ffffff" />
                        </svg>
                    </div>

                    <!-- Foto -->
                    <div class="id-avatar">
                        <img v-if="foto" :src="foto" alt="Foto" />
                        <i v-else class="pi pi-user"></i>
                    </div>

                    <!-- Datos -->
                    <div class="id-body">
                        <div class="id-name">
                            {{ tarjeta.nombre }}<br />
                            {{ tarjeta.paterno }} {{ tarjeta.materno }}
                        </div>
                        <div class="id-badge">{{ tarjeta.puesto || 'Empleado' }}</div>

                        <div class="id-info">
                            <div>
                                <span>RFC / ID</span>
                                <strong>{{ tarjeta.rfc }}</strong>
                            </div>
                            <div>
                                <span>Nacimiento</span>
                                <strong>{{ tarjeta.fechaTxt }}</strong>
                            </div>
                        </div>

                        <div class="id-qr">
                            <QrcodeVue
                                :value="`${tarjeta.nombre} ${tarjeta.paterno} ${tarjeta.materno} | ${tarjeta.rfc}`"
                                :size="110"
                                level="H"
                            />
                        </div>
                        <div class="id-scan">Escanea para verificar</div>
                    </div>

                    <!-- Pie -->
                    <div class="id-footer">Universidad Autónoma de Chiapas</div>
                </div>

                <div v-else class="text-gray-400">Llena el formulario y presiona GENERAR.</div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.id-card {
    position: relative;
    width: 300px;
    background: #ffffff;
    border-radius: 24px;
    overflow: hidden;
    box-shadow: 0 20px 45px rgba(15, 23, 42, 0.28);
    text-align: center;
    font-family: inherit;
}

/* Encabezado */
.id-top {
    position: relative;
    height: 150px;
    background: linear-gradient(135deg, #0f172a 0%, #1e3a8a 55%, #2563eb 100%);
    color: #fff;
    overflow: hidden;
}
.circle {
    position: absolute;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.08);
}
.c1 { width: 140px; height: 140px; top: -50px; right: -40px; }
.c2 { width: 90px; height: 90px; bottom: 10px; left: -30px; }

.id-brand {
    position: relative;
    z-index: 2;
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 18px 20px;
    text-align: left;
}
.id-logo {
    width: 42px;
    height: 42px;
    border-radius: 12px;
    background: linear-gradient(135deg, #22c55e, #16a34a);
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 800;
    letter-spacing: 1px;
}
.id-brand-name { font-size: 12px; font-weight: 700; letter-spacing: 1px; }
.id-brand-sub { font-size: 11px; opacity: 0.8; letter-spacing: 3px; }
.id-wave {
    position: absolute;
    bottom: -1px;
    left: 0;
    width: 100%;
    height: 50px;
}

/* Avatar */
.id-avatar {
    position: relative;
    z-index: 3;
    width: 112px;
    height: 112px;
    margin: -62px auto 0;
    border-radius: 50%;
    padding: 5px;
    background: linear-gradient(135deg, #22c55e, #2563eb);
    box-shadow: 0 10px 25px rgba(37, 99, 235, 0.35);
}
.id-avatar img,
.id-avatar i {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background: #eef2ff;
    display: flex;
    align-items: center;
    justify-content: center;
    object-fit: cover;
    font-size: 48px;
    color: #1e3a8a;
}

/* Cuerpo */
.id-body { padding: 14px 22px 8px; }
.id-name {
    font-size: 20px;
    font-weight: 800;
    line-height: 1.2;
    color: #0f172a;
    text-transform: uppercase;
}
.id-badge {
    display: inline-block;
    margin-top: 10px;
    padding: 4px 16px;
    border-radius: 999px;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 1px;
    color: #fff;
    background: linear-gradient(90deg, #16a34a, #2563eb);
}
.id-info {
    display: flex;
    justify-content: space-between;
    gap: 8px;
    margin-top: 16px;
    padding: 10px 12px;
    background: #f1f5f9;
    border-radius: 14px;
    text-align: left;
}
.id-info span {
    display: block;
    font-size: 10px;
    color: #64748b;
    text-transform: uppercase;
    letter-spacing: 1px;
}
.id-info strong { font-size: 12px; color: #0f172a; }

.id-qr {
    display: inline-block;
    margin-top: 16px;
    padding: 10px;
    background: #fff;
    border: 2px solid #e2e8f0;
    border-radius: 16px;
    box-shadow: 0 6px 16px rgba(15, 23, 42, 0.08);
}
.id-qr :deep(canvas) { display: block; }
.id-scan {
    margin-top: 6px;
    font-size: 10px;
    color: #94a3b8;
    letter-spacing: 1px;
    text-transform: uppercase;
}

/* Pie */
.id-footer {
    margin-top: 12px;
    padding: 10px;
    font-size: 11px;
    letter-spacing: 2px;
    text-transform: uppercase;
    color: #fff;
    background: linear-gradient(90deg, #0f172a, #1e3a8a, #16a34a);
}
</style>