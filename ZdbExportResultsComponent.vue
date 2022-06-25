<template>
    <div v-if="item.id">
        <div class="card-body">
            <div class="row">
                <div class="col-12 mb-3">
                    <h4 class="mb-0">Экспорт ответов</h4>
                </div>

                <template v-if="!disabled">
                    <div class="col-4">
                        <date-range-picker ref="picker" :opens="'inline'" :locale-data="localeData"
                            :single-date-picker="'range'" :timePicker="false" :auto-apply="true"
                            v-model="filter.dateRange">
                        </date-range-picker>
                    </div>
                    <div class="col-8">
                        <h5>Сегменты замера</h5>
                        <ul id="width_segments">
                            <li v-for="item in items" :key="item.message">
                                {{ item.message }}
                            </li>
                        </ul>
                    </div>

                    <div class="col-12 my-2">
                        <div class="float-right">
                            <span style="font-size: 16px;" class="mr-3" v-if="loadingCount !== null">
                                Количество ответов которые попадут в отчет:
                                <span v-if="loadingCount === false && resultsCount !== null">{{ resultsCount }}</span>
                                <span v-else>
                                    <img src="/img/circle-loader.svg" width="20" />
                                </span>
                            </span>
                            <span style="font-size: 16px; text-muted" class="mr-3" v-else>
                                Начните выбирать параметры фильтра для подсчета ответов
                            </span>
                            <button class="btn btn-sm"
                                :class="{ 'btn-primary': creatingExport === false, 'btn-light': creatingExport === true }"
                                :disabled="creatingExport === true" @click.prevent="createExport()">
                                <span v-if="creatingExport == false">Выгрузить</span>
                                <span v-else>
                                    <img src="/img/circle-loader.svg" width="15" /> Выгружаем...
                                </span>
                            </button>
                        </div>
                    </div>
                </template>

                <div class="col-12 mt-2">
                    <div class="card-body p-0">
                        <table class="table table-hover">
                            <tbody v-if="items.data && items.data.length > 0">
                                <tr>
                                    <th>ID</th>
                                    <th>Дата</th>
                                    <th>Статус</th>
                                    <th>Параметры</th>
                                    <th>Файл</th>
                                    <th>Кол-во ответов</th>
                                    <th>
                                        Действия
                                    </th>
                                    <th>
                                        <button class="btn btn-sm btn-success" @click.prevent="loadExports()"
                                            title="Обновить список экспортов">
                                            <i class="fas fa-sync-alt"></i>
                                        </button>
                                    </th>
                                </tr>
                                <tr v-for="item in items.data" :key="item.id">
                                    <td>{{ item.id }}</td>
                                    <td>
                                        <div>{{ prettyDate(item.created_at) }}</div>
                                    </td>
                                    <td>
                                        <span class="badge badge-primary" style="font-size: 14px;"
                                            v-if="item.status == 'wait'">
                                            В очереди
                                        </span>
                                        <span class="badge badge-warning" style="font-size: 14px;"
                                            v-else-if="item.status == 'active'">
                                            Выполняется
                                        </span>
                                        <span class="badge badge-success" style="font-size: 14px;"
                                            v-else-if="item.status == 'done'">
                                            Выполнен
                                        </span>
                                        <span class="badge badge-danger" style="font-size: 14px;"
                                            v-else-if="item.status == 'error'">
                                            Ошибка
                                        </span>
                                        <span class="badge badge-info" style="font-size: 14px;" v-else>
                                            {{ item.status }}
                                        </span>
                                        <!-- <div v-if="item.status == 'error' && item.error_message.length" class="text-danger">
                                            {{ item.error_message }}
                                        </div> -->
                                    </td>
                                    <td>
                                        Дата с: {{ prettyDate(item.params.dateRange.startDate) }}<br />
                                        Дата по: {{ prettyDate(item.params.dateRange.endDate) }}<br />
                                    </td>
                                    <td>
                                        <span v-if="item.filename">
                                            <a :href="'#'" @click.prevent="downloadExport(item.id)">Скачать</a>
                                        </span>
                                        <span v-else>
                                            &mdash;
                                        </span>
                                    </td>
                                    <td>
                                        <span v-if="item.results_number">
                                            {{ item.results_number }}
                                        </span>
                                        <span v-else>
                                            &mdash;
                                        </span>
                                    </td>
                                    <td>
                                        <!-- <button class="btn btn-sm btn-primary" @click.prevent="duplicateExport(item.id)" v-if="item.status != 'wait'">
                                            <i class="fas fa-sync-alt"></i> Повторить
                                        </button> -->
                                        <button class="btn btn-sm btn-danger" v-if="item.status != 'done'"
                                            @click.prevent="deleteExport(item.id)">
                                            Удалить
                                        </button>
                                    </td>
                                    <td> </td>
                                </tr>
                            </tbody>
                            <tbody v-else>
                                <tr class="table-warning">
                                    <td align="center">
                                        Экспорты ответов не найдены
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                    <div class="card-footer" v-if="items.data && items.data.length > 0">
                        <div class="row">
                            <div class="col-8">
                                <pagination :data="items" :limit="2" @pagination-change-page="changePage"></pagination>
                            </div>
                            <div class="col-4">
                                <div class="row">
                                    <div class="col-6">
                                        <span class="pr-3" v-show="items.total > 0">{{ items.from }} - {{ items.to }} из
                                            {{ items.total }}</span>
                                    </div>
                                    <!-- <div class="col-6">
                                        <select class="form-control form-control-sm" v-model="filter.limit">
                                            <option v-for="limit in limits" :key="limit">{{limit}}</option>
                                        </select>
                                    </div> -->
                                </div>
                            </div>
                        </div>
                    </div>
                    <div ref="loadingContainer" id="loadingContainer"></div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import DateRangePicker from 'vue2-daterange-picker';
//you need to import the CSS manually
import 'vue2-daterange-picker/dist/vue2-daterange-picker.css';

import * as appFunctions from "../../plugins/function";
import LoadingMixin from "../../mixins/loading";
import { DateTime as LuxonDateTime } from 'luxon';

var example1 = new Vue({
    el: '#width_segments',
    data: {
        items: [
            { message: '14-17' },
            { message: '17-24' },
            { message: '24-35' },
            { message: '35-44' },
        ]
    }
})


export default {
    props: {
        item: {
            type: Object,
            required: true
        },
        disabled: {
            type: Boolean,
            required: false
        }
    },
    mixins: [LoadingMixin],
    components: { DateRangePicker },
    data: function () {
        return {
            items: {},
            resultsCount: null,
            loadingCount: null,
            dateType: 'range',
            creatingExport: false,
            url: '/api/zdb-measure-results-export',
            filter: {
                dateRange: {
                    startDate: null,
                    endDate: null
                },
                page: 1,
                limit: 10,
            },
            localeData: {
                direction: 'ltr',
                format: 'dd.mm.yyyy',
                separator: ' - ',
                applyLabel: 'Применить',
                cancelLabel: 'Отмена',
                weekLabel: 'Н',
                customRangeLabel: 'Свой разброс',
                daysOfWeek: ['Вс', 'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб'],
                monthNames: ['Янв', 'Фев', 'Мар', 'Апр', 'Май', 'Июн', 'Июл', 'Авг', 'Сен', 'Окт', 'Ноя', 'Дек'],
                firstDay: 1
            }
        }
    },
    mounted() {
        this.filter.dateRange.startDate = this.item.lastPeriod.date_start;
        this.filter.dateRange.endDate = this.item.lastPeriod.date_stop;

        this.loadExports();
    },
    methods: {
        prettyDate: function (date) {
            date = date.replace(' ', 'T');
            if (!date.includes('Z')) {
                date = date + 'Z';
            }

            if (date && date.length > 0) {
                return LuxonDateTime.fromISO(date).toLocaleString(LuxonDateTime.DATE_FULL)
            }
            return '-';
        },
        changePage(page) {
            //this.isClickPage = true;
            this.filter.page = page;
            this.loadExports();
        },
        loadSegments: function () {


        },
        createExport: function () {
            this.creatingExport = true;
            axios.post(this.url + '/create', {
                measure_id: this.item.id,
                filter: this.filter,
                responseType: 'blob'
            }).then(response => {
                this.creatingExport = false;
                if (response && response.status === 200) {
                    this.downloadExport(response.data.export.id);
                    this.loadExports();
                }
            }).catch(error => {
                this.creatingExport = false;
                console.warn("createExport caught exception: ", error);
            });
        },
        preCountResults: function () {
            this.loadingCount = true;
            axios.post(this.url + '/count', {
                measure_id: this.item.id,
                filter: this.filter
            }).then(response => {
                this.loadingCount = false;
                if (response && response.status === 200) {
                    this.resultsCount = response.data.count;
                }
            }).catch(error => {
                this.loadingCount = false;
                console.warn("preCountResults caught exception: ", error);
            });
        },
        loadExports: function () {
            this.startLoading();
            axios.get(this.url + '/list?measure_id=' + this.item.id + "&page=" + this.filter.page)
                .then(response => {
                    if (response && response.status === 200) {
                        this.items = response.data;
                    }
                    this.stopLoading();
                }).catch(error => {
                    console.log("ERROR: ", error);
                    this.stopLoading();
                });
        },
        deleteExport: function (id) {
            swal.fire({
                title: 'Вы уверены?',
                showCancelButton: true,
                cancelButtonText: 'Отмена',
                confirmButtonText: 'Да',
                allowOutsideClick: false
            }).then((result) => {
                if (result.isConfirmed) {
                    axios.get(this.url + '/delete/' + id)
                        .then(response => {
                            this.loadExports();
                        });
                }
            });
        },
        downloadExport: function (id) {
            axios.get(this.url + '/download/' + id, {
                responseType: 'blob',
            }).then(response => {
                appFunctions.downloadFile(response);
            });
        },
    },
    watch: {
        'filter': {
            deep: true,
            immediate: false,
            handler: function (newVal) {
                this.preCountResults();
            }
        }
    }
}
</script>
