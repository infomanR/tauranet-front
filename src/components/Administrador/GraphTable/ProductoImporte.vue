<template>
    <div class="col-md-12">
        <div class="row">
            <div class="col-md-12">
                <h4 class="sub-cajero">Producto vs Importe</h4>
            </div>
        </div>
        <div class="row">
            <div class="col-md-3">
                <div class="form-group">
                    <flat-pickr
                            v-model="fecha.ini"                                                       
                            class="form-control input-style"
                            :config="config"
                            placeholder="Fecha Ini">
                    </flat-pickr>
                    <ListErrors :errores="errores.ini"></ListErrors>
                </div>
            </div>
            <div class="col-md-3">
                <div class="form-group">
                    <flat-pickr
                        v-model="fecha.fin"                                                       
                        class="form-control input-style"
                        :config="config"
                        placeholder="Fecha Fin"
                    >
                    </flat-pickr>
                    <ListErrors :errores="errores.fin"></ListErrors>
                </div>
            </div>
            <div class="col-md-3">
                <div class="form-group">
                    <label for="">
                        <select class="form-control input-style" v-model="comboCategorias">
                            <option value="-1" selected>Categoria...</option>
                            <option v-for="s in listaCategoria" v-bind:key="s.id_categoria_producto" :value="s.id_categoria_producto">{{s.nombre}}</option>
                        </select>
                    </label>
                </div>
            </div>
            <div class="col-md-3">
                <button @click="productoImporteMethod()" class="btn btn-primary float-right" ref="btnBuscarRef">Buscar</button>
            </div>
        </div>
        <div class="row" v-if="loaded">
            <div class="col-md-8 table-responsive">
                <BarChart :chart-data="empleadoImporteCollection" :options="options"></BarChart>
            </div>
            <div class="col-md-4">
                <div class="row">
                    <div class="col-md-12 table-responsive">
                        <table-component
                            :data="productoImporteTableArray"
                            tableClass="table table-bordered table-condensed"
                            theadClass="head-table"
                            filterPlaceholder="Buscar..."
                            filter-input-class="inputSearchText"
                            :show-caption=false
                            >
                            <table-column show="nom_producto" label="Producto"></table-column>
                            <table-column show="nom_categoria" label="Categoría"></table-column>
                            <table-column show="importe" label="Importe"></table-column>
                        </table-component>
                    </div>
                </div>
                <div class="row">
                    <div class="col-md-12 table-responsive">
                        <Pagination :pagination="pagination" v-on:funcion="productoImporteMethod"></Pagination>
                    </div>
                </div>
            </div>
        </div>
    </div>  
</template>
<script>
const axios = require("axios");
import flatPickr from 'vue-flatpickr-component';
import BarChart from '@/components/Charts/BarChart';
import Pagination from '@/components/Pagination/Pagination';
import 'flatpickr/dist/flatpickr.css';
import ListErrors from '@/components/Messages/ListErrors';
export default{
    name: 'ProductoImporte',
    components: {
        flatPickr,
        BarChart,
        Pagination,
        ListErrors
    },
    data() {
        return {
            comboCategorias: -1,
            listaCategoria: [],
            loaded: false,
            productoImporteArray: [],
            productoImporteTableArray: [],
            pagination: {},
            empleadoImporteCollection: null,
            labels: [],
            importe: [],
            errores: {},
            fecha: {
                ini: null,
                fin: null
            },
            config: {
                wrap: true, // set wrap to true only when using 'input-group'
                altFormat: 'j/n/Y',
                altInput: true,
                dateFormat: 'Y-m-d',
                allowInput: true,
                wrap: false,
                static: true     
            },
            options: {
                responsive: true,
                maintainAspectRatio: false
            }
        }
    },
    created () {
        this.comboCategoria()
    },
    methods: {
        comboCategoria(){
            this.$Progress.start()
            axios.defaults.headers.common["Authorization"] = "Bearer " + this.$store.state.token;
                axios.get(this.$store.state.url_root+`api/auth/cproducto/restaurant/${this.$store.state.id_restaurant}`)
            .then(response => {
                this.listaCategoria = response.data.data;
                this.$Progress.finish()
            })
            .catch (error => {
                this.$toasted.show("Productos.vue: "+error, {type: 'error'})
                this.$Progress.fail()
            });
        },
        productoImporteMethod(url) {
            this.$Progress.start()
            this.$refs.btnBuscarRef.className = "btn btn-primary float-right disabled"
            url = url || this.$store.state.url_root+`api/auth/productoimporte/${this.$store.state.id_restaurant}/categoria/${this.comboCategorias}/fechaini/${this.fecha.ini}/fechafin/${this.fecha.fin}`
            this.labels = []
            this.importe = []
            axios.defaults.headers.common["Authorization"] = "Bearer " + this.$store.state.token;
            axios.get(url)
            .then(response => {
                if(response.data.error == null){
                    this.errores = {}
                    this.productoImporteArray = response.data.data;
                    this.productoImporteTableArray = response.data.dataT.data
                    this.pagination = response.data.dataT
                    this.productoImporteArray.forEach(element => {
                        this.labels.push(element.nom_producto)
                        this.importe.push(element.importe)
                    })
                    this.empleadoImporteCollection = {
                        labels: this.labels,
                        datasets: [{
                            label: 'Producto',
                            backgroundColor: '#ffaa00',
                            data: this.importe
                        }]
                    }
                    this.loaded = true
                    this.$refs.btnBuscarRef.className = "btn btn-primary float-right"
                    this.$Progress.finish()
                }else{
                    this.errores = response.data.error
                    this.$refs.btnBuscarRef.className = "btn btn-primary float-right"
                    this.$Progress.fail()
                }
            })
            .catch (error => {
                this.$toasted.show("ProductoImporte.vue: "+error, {type: 'error'})
                this.$Progress.fail()
                this.$refs.btnBuscarRef.className = "btn btn-primary float-right"
            });
        }
    },
}
</script>