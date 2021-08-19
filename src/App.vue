<template>
    <div id="app">
        
        <b-card no-body class="mb-1">
            <b-card-header header-tag="header" class="p-1" role="tab">
                <b-button block href="#" v-b-toggle.accordion-1 variant="danger">BI Selection</b-button> <strong>{{ biSelection }}</strong> 
            </b-card-header>
            <b-collapse id="accordion-1" visible accordion="my-accordion" role="tabpanel">
                <b-card-body>
                
                    <b-form-select v-model="biSelection" :options="biSelections" @change="changeBISelection($event)"></b-form-select>       
                
                </b-card-body>
            </b-collapse>
        </b-card>

        <b-card no-body class="mb-1">
            <b-card-header header-tag="header" class="p-1" role="tab">
                <b-button block href="#" v-b-toggle.accordion-2 variant="warning">Options</b-button>
            </b-card-header>
            <b-collapse id="accordion-2" accordion="my-accordion" role="tabpanel">
                <b-card-body>
                    <div class="button-right">
                        <b-button block variant="dark"v-b-modal.modal-approve>Approve</b-button>

                        <b-modal id="modal-approve" title="Partner Stock Approval" @ok="approveBISelection">
                            <p class="my-4">Are you sure you want to Approve <span class="high-light">{{biSelection}}</span>?</p>                            
                        </b-modal>
                   </div>

                    <ag-grid-vue 
                        style="width: 100%; height: 800px;"
                        class="ag-theme-alpine"
                        id="myGrid"
                        :gridOptions="gridOptions"
                        @grid-ready="onGridReady"
                        :columnDefs="columnDefs"                        
                        :defaultColDef="defaultColDef"
                        :suppressRowClickSelection="true"                       
                        :debug="true"
                        :rowSelection="rowSelection"                               
                        :enableRangeSelection="true"
                        :pagination="true"
                        :paginationAutoPageSize="true"                    
                        :floatingFilter="true"
                        :rowClassRules="rowClassRules"
                        :rowData="optionData">
                    </ag-grid-vue>

                </b-card-body>
            </b-collapse>
        </b-card>

        <b-card no-body class="mb-1">
            <b-card-header header-tag="header" class="p-1" role="tab">
                <b-button block href="#" v-b-toggle.accordion-3 variant="success" v-on:click="updateSelected">Sizes
                    <b-spinner small type="grow" :hidden="isLoaded"></b-spinner>
                </b-button> 

           </b-card-header>
            <b-collapse id="accordion-3" accordion="my-accordion" role="tabpanel">
                <b-card-body>
                    
                    <b-table ref="table" striped hover sticky-header :items="optionSelection" :fields="fields" @input="tableLoaded">

                        <template v-slot:cell()="{ item, field: { key } }">
                            <b-form-input :class="item['StyleOption']+item['Store'].replace(' ','-')" type="number" v-if="key.substring(0,1)==='$'" v-model="item[key]" v-on:change="updateSizes(item['BISelection'],item['StyleOption'],item['Store'],key,$event,item['StyleOption']+item['Store'].replace(' ','-'))"></b-form-input>
                            <b-form-input type="number" v-else-if="key==='Split'" v-model="item[key]"></b-form-input>
                            <span v-else>{{item[key]}}</span>
                        </template>

                    </b-table>
                
                </b-card-body>
            </b-collapse>
        </b-card>

    </div>
</template>

<script>
   import { AgGridVue } from "ag-grid-vue";
   import axios from 'axios'

   export default {
       name: 'App',

       components: {
            AgGridVue,
        },

        data() {
            return {
                gridOptions: null,
                gridApi: null,
                columnApi: null,
                columnDefs: null,            
                defaultColDef: null,
                rowSelection: null,           
                biSelections: null,
                biSelection: null,
                optionData: null,
                optionSelection: [],
                items: null,
                isLoaded: true,
                fields: [],
                selectedRow: {},
                selectedCell: null                   
            }
        },

        beforeMount() {          

            this.gridOptions = {};

            this.columnDefs = [
             
              { field: 'category', 
                minWidth: 70,
                checkboxSelection: checkboxSelection,
                headerCheckboxSelection: headerCheckboxSelection,    
                headerCheckboxSelectionFilteredOnly: true,   
                sortable: true, 
                filter: true, 
                headerName: "Category",
                suppressMenu: true,
                },   
              { field: 'styleOption', sortable: true, filter: true, headerName: "Option", suppressMenu: true, },  
              { field: 'styleDesc', sortable: true, filter: true, headerName: "Style", suppressMenu: true, },  
              { field: 'store', sortable: true, filter: true, headerName: "Store", suppressMenu: true, },                                        
            ];

            this.defaultColDef = {
       
            sortable: true,
            resizable: true,
            filter: true,
            flex: 1,
            minWidth: 100,
            };
            this.rowSelection = 'multiple';
            
            fetch('https://localhost:5001/api/ist/v1/bi-selection?partner=jl')
                .then(result => result.json())
                .then(biSelections => this.biSelections = biSelections);

              this.rowClassRules = {
                'dirty-row': (params) => {
                    var dirty = params.data.dirty;
                    return dirty >0;
                }
              }
            
        },
        async mounted() {
            this.gridApi = this.gridOptions.api;
            this.gridColumnApi = this.gridOptions.columnApi;
        },

        methods: { 

            async updateSizes(selection,option,store,size,value,className)
            {         
            
                 const request = new Request(
                    "https://localhost:5001/api/ist/v1/partner-stock/",
                    {
                    method: "POST",
                    headers: {
                         'Content-Type': 'application/json'
                     },
                    body: '['+JSON.stringify({ action: 'updateSize', selection: selection, option: option, store: store, size: size.replace('$',''), value: value })+']'
                    }
                );

                const res = await fetch(request);
                const data = await res.json();
                if(data===1)
                {
                    var total = 0; 
                    
                    var elements =  $("input[class*='"+className+"']");

                    $.each(elements, function(index, value)
                    {
                        total= total + (value.value)*1;
                        value.classList.remove('warning');
                    });

                   var url = 'https://localhost:5001/api/ist/v1/partner-stock/validate/'+option+'/'+total+'/size='+size.replace('$','')+'';
                  
                    const valres = await fetch(url);
                    const valdata = await valres.json();
                   
                  
                    if(!valdata)
                    {
                        $.each(elements, function(index, value)
                        {
                            value.classList.add('warning');
                        });
                    }
                                          
                    
                }
            },

            async tableLoaded() {

                while(!document.querySelector(".table-b-table-default")) {
                     await new Promise(r => setTimeout(r, 500));
                 }
  
                var headers = this.$refs.table.$el.querySelectorAll('thead tr th div');

                if(headers.length>0)
                {
                    $.each(headers, function(index, value)
                    {
                        if(value.innerHTML.substring(0,1)==='$' || value.innerHTML==='Total')
                        {
                            value.innerHTML = '';
                        }
                    });
                }

            },

            async changeBISelection (selection) {
                var url = 'https://localhost:5001/api/ist/v1/partner-stock-option/selection='+selection

               await fetch(url)
                    .then(result => result.json())
                    .then(optionData => this.optionData = optionData);

            },

             async approveBISelection () {
             
                 const request = new Request(
                    "https://localhost:5001/api/ist/v1/partner-stock/",
                    {
                    method: "POST",
                    headers: {
                         'Content-Type': 'application/json'
                     },
                    body: '['+JSON.stringify({ action: 'approveSelection', selection: this.$data.biSelection })+']'
                    }
                );

                const res = await fetch(request);
                const data = await res.json();

            },
        
            onGridReady(params) {
                    const updateData = (data) => {
                    this.rowData = optionData;
                };               
            },

            async updateSelected()
            {     
    
                this.$data.isLoaded = false;

                const selectedNodes = this.gridApi.getSelectedNodes();
                const selectedData = selectedNodes.map( node => node.data );
                              
                await this.getOptionSizes(selectedData)
               
                 while(!document.querySelector(".table-b-table-default")) {
                     await new Promise(r => setTimeout(r, 500));
                 }
                 this.$data.isLoaded = true
            
            },

           async getOptionSizes(selectedData)
            {

                var selection = this.$data.biSelection;
                var rtnData = [];
                var fields = [];

                $.each(selectedData, async function(index, value) {
                    
                   var payload = JSON.stringify(value).replace('{','{"selection":"'+selection+'",');

                   var url = "https://localhost:5001/api/ist/v1/partner-stock/["+payload+"]";
                   let response = await fetch(url);
                   let data = await response.json();

                    $.each(data, function(index, value)
                    {
                        $.each(value, function(index, value)
                        {
                            if(index !=='PartnerStockId' && index !=='BISelection' && index !=='Dirty')
                            {
                                if($.inArray(index, fields) === -1)
                                {          
                                    switch(index)
                                    {
                                        case "StyleOption":
                                        fields.push({key: index, sortable: true, label: 'Option'});
                                        break;

                                        case "S5Category":
                                        fields.push({key: index, sortable: true, label: 'Cat'});
                                        break;

                                        case "S5Department":
                                        fields.push({key: index, sortable: true, label: 'Dept'});
                                        break;
                                    
                                        case "StyleDesc":
                                        fields.push({key: index, sortable: true, label: 'Style'});
                                        break;

                                        case "Store":
                                        fields.push({key: index, sortable: true});
                                        break;

                                        case "Total":
                                        fields.push({key: index, sortable: false});
                                        break;

                                        case "Split":
                                        fields.push({key: index, sortable: false});
                                        break;

                                        default:
                                        fields.push({key: index, sortable: false, thClass: 'text-right', tdClass: 'text-right'});

                                    }                         
                                    
                                }
                            }
                        });
                        rtnData.push(value);                    
                    });
               });
            
               this.$data.fields = await fields;
               this.$data.optionSelection= await rtnData;
                                    
            }   
        },
        
   }

    function checkboxSelection (params) {
        return params.columnApi.getRowGroupColumns().length === 0;
    }

    function headerCheckboxSelection  (params) {
        return params.columnApi.getRowGroupColumns().length === 0;
    }

</script>

<style lang="scss">
  @import "../node_modules/ag-grid-community/dist/styles/ag-grid.css";
  @import "../node_modules/ag-grid-community/dist/styles/ag-theme-alpine.css";

   .btn-block{
      width: 160px;
      margin-bottom: 5px;
  }

  .b-table-sticky-header
  {
      max-height: 700px!important;
      min-width: 1900px!important;
  }

  .b-table input
  {
      width: 28px!important;
      padding: 0;
      text-align: right;
  }

   .b-table td
  {
      padding: 0.1rem 0.1rem!important;
  }
  
  input[type='number'] {
    -moz-appearance:textfield;
  }

  input::-webkit-outer-spin-button,
  input::-webkit-inner-spin-button {
    -webkit-appearance: none;
  }

span.sr-only
{
    display: none;
}

.text-right
{
    text-align: right;
}

.warning{
    color: red!important;
    background-color: yellow!important;
}

.button-right
{
    text-align: right;
}

.dirty-row{
    background-color: pink!important;
    //border: thin solid pink!important;
}

.high-light
{
    font-weight: bold;
}
</style>