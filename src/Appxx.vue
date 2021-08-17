<template>
  <div id="app">
    <b-card no-body class="mb-1">
      <b-card-header header-tag="header" class="p-1" role="tab">
        <b-button block href="#" v-b-toggle.accordion-1 variant="info">BI Selection</b-button> <strong>{{ biSelection }}</strong>
      </b-card-header>
      <b-collapse id="accordion-1" visible accordion="my-accordion" role="tabpanel">
        <b-card-body>
          
            <b-form-select v-model="biSelection" :options="biSelections" @change="changeJobTitle($event)"></b-form-select>           
          
        </b-card-body>
      </b-collapse>
    </b-card>
    <b-card no-body class="mb-1">
      <b-card-header header-tag="header" class="p-1" role="tab">
        <b-button block href="#" v-b-toggle.accordion-2 variant="info">Options</b-button>
      </b-card-header>
      <b-collapse id="accordion-2" accordion="my-accordion" role="tabpanel">
        <b-card-body>
            <b-button variant="success" v-b-modal.modal-1 @click="getSelectedRows()">Allocation</b-button>

            <!--div class="overflow-auto">
                <b-pagination
                v-model="currentPage"
                :total-rows="rows"
                :per-page="perPage"
                aria-controls="my-table"
                ></b-pagination>

                <p class="mt-3">Current Page: {{ currentPage }}</p>

             <vs-pagination
                 :total="totalPages"
                 :max="10"
                 v-model="currentPage" />

                 :paginationAutoPageSize=true

            </div-->
            
            <ag-grid-vue style="width: 1860px; height: 800px;"
                class="ag-theme-alpine"
                :columnDefs="columnDefs"
                :rowData="optionData"
                :animateRows="true"
                :floatingFilter="true"
            :pagination="true"
                :paginationPageSize="paginationPageSize"
                :paginationNumberFormatter="paginationNumberFormatter"
                rowSelection="multiple"
                @grid-ready="onGridReady">
            </ag-grid-vue>


         
        </b-card-body>
      </b-collapse>
    </b-card>
    <b-card no-body class="mb-1">
      <b-card-header header-tag="header" class="p-1" role="tab">
        <b-button block href="#" v-b-toggle.accordion-3 variant="info">Update</b-button>
      </b-card-header>
      <b-collapse id="accordion-3" accordion="my-accordion" role="tabpanel">
        <b-card-body>
          <b-card-text>accordion 3 text</b-card-text>
        </b-card-body>
      </b-collapse>
    </b-card>
  </div>
</template>

<script>
    import { AgGridVue } from "ag-grid-vue";

    export default {
        name: 'App',

        components: {
            AgGridVue,
        },


        methods: {            
            changeJobTitle (selection) {
                var url = 'https://localhost:5001/api/ist/v1/partner-stock-option/selection='+selection

                fetch(url)
                    .then(result => result.json())
                    .then(optionData => this.optionData = optionData);

            },
            onGridReady(params) {
                const updateData = (data) => {
                this.rowData = data;
                params.api.paginationGoToPage(4);
                };
            },
            getSelectedRows() {
                if(this.gridApi.getSelectedNodes().length===0)
                {                 
                    this.gridApi.selectAllFiltered();                                  
                }
                const selectedNodes = this.gridApi.getSelectedNodes();
                const selectedData = selectedNodes.map( node => node.data );

                alert(JSON.stringify(selectedData));
            },
        },

        data() {
            return {
                biSelections: null,
                biSelection: null,
                optionData: null,   

                paginationNumberFormatter: null,
            }
        },

        beforeMount() {

            this.paginationPageSize = 10;
            this.paginationNumberFormatter = (params) => {
                return '[' + params.value.toLocaleString() + ']';
            };

             this.columnDefs = [
             
              { field: 'category', checkboxSelection: true, sortable: true, filter: true, headerName: "Category" },   
              { field: 'styleOption', sortable: true, filter: true, headerName: "Option" },  
              { field: 'styleDesc', sortable: true, filter: true, headerName: "Style" },  
              { field: 'store', sortable: true, filter: true, headerName: "Store" },  
                        
            ];

            fetch('https://localhost:5001/api/ist/v1/bi-selection?partner=jl')
            .then(result => result.json())
            .then(biSelections => this.biSelections = biSelections);
           
        },
         mounted() {
            this.gridApi = this.gridOptions.api;
            this.gridColumnApi = this.gridOptions.columnApi;
        },
    }

</script>
<style lang="scss">
  @import "../node_modules/ag-grid-community/dist/styles/ag-grid.css";
  @import "../node_modules/ag-grid-community/dist/styles/ag-theme-alpine.css";

  .btn-block{
      width: 160px;
  }

</style>