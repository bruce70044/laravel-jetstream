<template>
  <app>
    <div class="p-grid crud-demo">
      <div class="p-col-12">
        <div class="card">
          <Toast />
          <Toolbar class="p-mb-4">
            <template v-slot:left>
              <Button
                label="Novo"
                icon="pi pi-plus"
                class="p-button-success p-mr-2"
                @click="openNew"
              />
              <Button
                label="Deletar"
                icon="pi pi-trash"
                class="p-button-danger"
                @click="confirmDeleteSelected"
                :disabled="!selectedRegs || !selectedRegs.length"
              />
            </template>

            <template v-slot:right>
              <Button
                label="Exportar"
                icon="pi pi-upload"
                class="p-button-info"
                @click="exportCSV($event)"
              />
            </template>
          </Toolbar>

          <DataTable
            ref="dt"
            :value="condpgtos"
            v-model:selection="selectedRegs"
            dataKey="id"
            :paginator="true"
            :rows="10"
            :loading="loadingTable"
            paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
            :rowsPerPageOptions="[5, 10, 25]"
            currentPageReportTemplate="Exibindo {first} até {last} de {totalRecords} condições de pagamento"
            responsiveLayout="scroll"
          >
            <template #header>
              <div
                class="table-header p-d-flex p-flex-column p-flex-md-row p-jc-md-between"
              >
                <h5 class="p-m-0">Condições de Pagamento</h5>
                <span class="p-input-icon-left">
                  <i class="pi pi-search" />
                  <InputText v-model="filter" placeholder="Pesquisar..." />
                </span>
              </div>
            </template>

            <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>
            <Column field="code" header="Código" :sortable="true">
              <template #body="slotProps">
                <span class="p-column-title">Código</span>
                {{ slotProps.data.codigo }}
              </template>
            </Column>
            <Column field="descricao" header="Descrição" :sortable="true">
              <template #body="slotProps">
                <span class="p-column-title">Descrição</span>
                {{ slotProps.data.descricao }}
              </template>
            </Column>
            <Column
              field="qtditens"
              header="Volume Mínimo"
              :sortable="true"
              class="p-text-right"
            >
              <template #body="slotProps">
                <span class="p-column-title">Volume Mínimo</span>
                {{ formatDecimal(slotProps.data.qtditens) }}
              </template>
            </Column>
            <Column
              field="valormin"
              header="Valor Mínimo"
              :sortable="true"
              class="p-text-right"
            >
              <template #body="slotProps">
                <span class="p-column-title">Valor Mínimo</span>
                {{ formatDecimal(slotProps.data.valormin) }}
              </template>
            </Column>

            <Column field="inventoryStatus" header="Status" :sortable="true">
              <template #body="slotProps">
                <span class="p-column-title">Status</span>
                <span
                  :class="
                    'condpgto-badge status-' +
                    (slotProps.data.status
                      ? slotProps.data.status.toLowerCase()
                      : '')
                  "
                  >{{ slotProps.data.status }}</span
                >
              </template>
            </Column>

            <Column
              field="activity"
              header="Participação"
              :showFilterMatchModes="false"
              style="min-width: 12rem"
            >
              <template #body="{ data }">
                <span class="p-column-title">Participação</span>
                <ProgressBar :value="data.id" :showValue="false"></ProgressBar>
              </template>
              <template #filter="{ filterModel }">
                <Slider
                  v-model="filterModel.value"
                  range
                  class="p-m-3"
                ></Slider>
                <div class="p-d-flex p-ai-center p-jc-between p-px-2">
                  <span>{{
                    filterModel.value ? filterModel.value[0] : 0
                  }}</span>
                  <span>{{
                    filterModel.value ? filterModel.value[1] : 100
                  }}</span>
                </div>
              </template>
            </Column>

            <Column>
              <template #body="slotProps">
                <Button
                  icon="pi pi-pencil"
                  class="p-button-rounded p-button-success p-mr-2"
                  @click="edit(slotProps.data)"
                />
                <Button
                  icon="pi pi-trash"
                  class="p-button-rounded p-button-danger"
                  @click="confirmarDelete(slotProps.data)"
                />
              </template>
            </Column>
          </DataTable>

          <Dialog
            v-model:visible="editDialog"
            :style="{ width: '450px' }"
            header="Editar"
            :modal="true"
            class="p-fluid"
          >
            <div class="p-field">
              <label for="descricao">Descrição</label>
              <InputText
                id="descricao"
                v-model.trim="condpgto.descricao"
                required="true"
                autofocus
                :class="{ 'p-invalid': submitted && !condpgto.nome }"
              />
              <small class="p-invalid" v-if="submitted && !condpgto.descricao"
                >Descrição é obrigatório.</small
              >
            </div>

            <div class="p-field">
              <label for="status" class="p-mb-3">Status</label>
              <Dropdown
                id="status"
                v-model="condpgto.status"
                :options="statuses"
                optionLabel="label"
                required="true"
                :class="{ 'p-invalid': submitted && !condpgto.status }"
                placeholder="Selecionar Status"
              >
                <template #value="slotProps">
                  <div v-if="slotProps.value && slotProps.value.value">
                    <span
                      :class="'condpgto-badge status-' + slotProps.value.value"
                      >{{ slotProps.value.label }}</span
                    >
                  </div>
                  <div v-else-if="slotProps.value && !slotProps.value.value">
                    <span
                      :class="
                        'condpgto-badge status-' + slotProps.value.toLowerCase()
                      "
                      >{{ slotProps.value }}</span
                    >
                  </div>
                  <span v-else>
                    {{ slotProps.placeholder }}
                  </span>
                </template>
              </Dropdown>
              <small class="p-invalid" v-if="submitted && !condpgto.status"
                >Status é obrigatório.</small
              >
            </div>

            <div class="p-formgrid p-grid">
              <div class="p-field p-col">
                <label for="qtditens">Volume Mínimo</label>
                <InputNumber id="qtditens" v-model="condpgto.qtditens" />
              </div>

              <div class="p-field p-col">
                <label for="valormin">Valor Mínimo</label>
                <InputNumber
                  id="valormin"
                  v-model="condpgto.valormin"
                  mode="currency"
                  currency="BRL"
                  locale="pt-BR"
                />
              </div>
            </div>
            <template #footer>
              <Button
                label="Cancelar"
                icon="pi pi-times"
                class="p-button-text"
                @click="hideDialog"
              />
              <Button
                label="Salvar"
                icon="pi pi-check"
                class="p-button-text"
                @click="save"
              />
            </template>
          </Dialog>

          <Dialog
            v-model:visible="showDeleteDialog"
            :style="{ width: '450px' }"
            header="Confirmar"
            :modal="true"
          >
            <div class="confirmation-content">
              <i
                class="pi pi-exclamation-triangle p-mr-3"
                style="font-size: 2rem"
              />
              <span v-if="condpgto"
                >Tem certeza de que deseja excluir <b>{{ condpgto.name }}</b
                >?</span
              >
            </div>
            <template #footer>
              <Button
                label="Não"
                icon="pi pi-times"
                class="p-button-text"
                @click="showDeleteDialog = false"
              />
              <Button
                label="Sim"
                icon="pi pi-check"
                class="p-button-text"
                @click="delete"
              />
            </template>
          </Dialog>

          <Dialog
            v-model:visible="deleteDialog"
            :style="{ width: '450px' }"
            header="Confirmar"
            :modal="true"
          >
            <div class="confirmation-content">
              <i
                class="pi pi-exclamation-triangle p-mr-3"
                style="font-size: 2rem"
              />
              <span v-if="condpgto"
                >Confirma deletar todos os registros selecionados?</span
              >
            </div>
            <template #footer>
              <Button
                label="Não"
                icon="pi pi-times"
                class="p-button-text"
                @click="deleteDialog = false"
              />
              <Button
                label="Sim"
                icon="pi pi-check"
                class="p-button-text"
                @click="deleteSelectedRegs"
              />
            </template>
          </Dialog>
        </div>
      </div>
    </div>
  </app>
</template>

<script>
import _ from "lodash";
import App from "@/Layouts/App";
import CondPgtosService from "./CondPgtosService";

export default {
  components: {
    App,
  },
  data() {
    return {
      condpgtos: null,
      editDialog: false,
      showDeleteDialog: false,
      deleteDialog: false,
      condpgto: {},
      selectedRegs: null,
      filter: "",
      submitted: false,
      loadingTable: true,
      statuses: [
        { label: "Ativo", value: "A" },
        { label: "Inativo", value: "I" },
      ],
    };
  },
  service: null,
  created() {
    this.service = new CondPgtosService();
    this.debouncedGet = _.debounce(this.getCondPgtos, 500);
  },
  mounted() {
    this.getCondPgtos();
  },
  watch: {
    filter: function (value) {
      this.debouncedGet();
    },
  },
  methods: {
    formatDecimal(value) {
      if (value) {
        var formatter = new Intl.NumberFormat("pt-BR", {
          style: "decimal",
          minimumFractionDigits: 2,
          currency: "BRL",
        });
        return formatter.format(value);
      }
      return 0;
    },

    openNew() {
      this.condpgto = {};
      this.submitted = false;
      this.editDialog = true;
    },

    hideDialog() {
      this.editDialog = false;
      this.submitted = false;
    },

    getCondPgtos() {
      this.service.getAll(this.filter).then((data) => {
        this.condpgtos = data;
        this.loadingTable = false;
      });
    },

    save() {
      this.submitted = true;
      if (this.condpgto.descricao && this.condpgto.status) {
        this.loadingTable = true;
        if (this.condpgto.id) {
          // Save editted condpgto
          this.condpgto.status = this.condpgto.status.value
            ? this.condpgto.status.value
            : this.condpgto.status;

          this.service.update(this.condpgto).then((data) => {
            this.condpgtos[this.findIndexById(data.id)] = data;
            this.$toast.add({
              severity: "success",
              summary: "Successo",
              detail: "Condição Atualizada",
              life: 3000,
            });
            this.editDialog = false;
            this.condpgto = {};
            this.loadingTable = false;
          });
        } else {
          this.condpgto.codigo = this.createCode();
          this.condpgto.image = "bamboo-watch.jpg"; // Use template image
          this.condpgto.status = this.condpgto.status
            ? this.condpgto.status.value
            : "A";
          this.service.store(this.condpgto).then((data) => {
            this.condpgtos.unshift(data);
            this.$toast.add({
              severity: "success",
              summary: "Successo",
              detail: "Condição Criada",
              life: 3000,
            });
            this.editDialog = false;
            this.condpgto = {};
            this.loadingTable = false;
          });
        }
      }
    },
    // Open edit condpgto modal
    edit(condpgto) {
      this.service.edit(condpgto.id).then((data) => {
        this.condpgto = { ...data };
        this.editDialog = true;
      });
    },
    // Open confirm delete condpgto modal
    confirmarDelete(condpgto) {
      this.condpgto = condpgto;
      this.showDeleteDialog = true;
    },
    // Delete condpgto
    delete() {
      this.service.delete([this.condpgto.id]).then((id) => {
        this.condpgtos = this.condpgtos.filter(
          (val) => val.id !== Number(id[0])
        );
        this.showDeleteDialog = false;
        this.condpgto = {};
        this.$toast.add({
          severity: "success",
          summary: "Successo",
          detail: "Condição Deletado",
          life: 3000,
        });
      });
    },
    // Find condpgto by id
    findIndexById(id) {
      let index = -1;
      for (let i = 0; i < this.condpgtos.length; i++) {
        if (this.condpgtos[i].id === id) {
          index = i;
          break;
        }
      }
      return index;
    },
    // Generate condpgto code
    createCode() {
      let code = "";
      var chars =
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
      for (var i = 0; i < 5; i++) {
        code += chars.charAt(Math.floor(Math.random() * chars.length));
      }
      return code;
    },

    exportCSV() {
      this.$refs.dt.exportCSV();
    },
    // Open confirm delete selected condpgtos
    confirmDeleteSelected() {
      this.deleteDialog = true;
    },
    // Delete selected condpgtos
    deleteSelectedRegs() {
      let ids = this.selectedRegs.map((item) => {
        return item.id;
      });

      this.service.delete(ids).then((id) => {
        this.condpgtos = this.condpgtos.filter(
          (val) => !id.includes(val.id.toString())
        );
        this.deleteDialog = false;
        this.selectedRegs = null;
        this.$toast.add({
          severity: "success",
          summary: "Successo",
          detail: "Condição Deletado",
          life: 3000,
        });
      });
    },
  },
};
</script>

<style scoped lang="scss">
.condpgto-image {
  width: 100px;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
}

.p-dialog .condpgto-image {
  width: 150px;
  margin: 0 auto 2rem auto;
  display: block;
}

.confirmation-content {
  display: flex;
  align-items: center;
  justify-content: center;
}

.condpgto-badge {
  border-radius: 2px;
  padding: 0.25em 0.5rem;
  text-transform: uppercase;
  font-weight: 700;
  font-size: 12px;
  letter-spacing: 0.3px;

  &.status-a {
    background: #c8e6c9;
    color: #256029;
  }

  &.status-i {
    background: #ffcdd2;
    color: #c63737;
  }

  &.status-x {
    background: #feedaf;
    color: #8a5340;
  }
}

::v-deep(.p-datatable-frozen-tbody) {
  font-weight: bold;
}

::v-deep(.p-datatable-scrollable .p-frozen-column) {
  font-weight: bold;
}

::v-deep(.p-progressbar) {
  height: 0.5rem;
  background-color: #d8dadc;

  .p-progressbar-value {
    background-color: #607d8b;
  }
}

.p-datatable .p-column-filter {
  display: none;
}

.table-header {
  display: flex;
  justify-content: space-between;
}

::v-deep(.p-datatable.p-datatable-customers) {
  .p-datatable-header {
    padding: 1rem;
    text-align: left;
    font-size: 1.5rem;
  }

  .p-paginator {
    padding: 1rem;
  }

  .p-datatable-thead > tr > th {
    text-align: left;
  }

  .p-datatable-tbody > tr > td {
    cursor: auto;
  }

  .p-dropdown-label:not(.p-placeholder) {
    text-transform: uppercase;
  }
}

/* Responsive */
.p-datatable-customers .p-datatable-tbody > tr > td .p-column-title {
  display: none;
}

.customer-badge {
  border-radius: 2px;
  padding: 0.25em 0.5rem;
  text-transform: uppercase;
  font-weight: 700;
  font-size: 12px;
  letter-spacing: 0.3px;

  &.status-qualified {
    background: #c8e6c9;
    color: #256029;
  }

  &.status-unqualified {
    background: #ffcdd2;
    color: #c63737;
  }

  &.status-negotiation {
    background: #feedaf;
    color: #8a5340;
  }

  &.status-new {
    background: #b3e5fc;
    color: #23547b;
  }

  &.status-renewal {
    background: #eccfff;
    color: #694382;
  }

  &.status-proposal {
    background: #ffd8b2;
    color: #805b36;
  }
}

.p-progressbar-value.ui-widget-header {
  background: #607d8b;
}

@media (max-width: 640px) {
  .p-progressbar {
    margin-top: 0.5rem;
  }
}

.product-image {
  width: 100px;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
}

.orders-subtable {
  padding: 1rem;
}

.product-badge {
  border-radius: 2px;
  padding: 0.25em 0.5rem;
  text-transform: uppercase;
  font-weight: 700;
  font-size: 12px;
  letter-spacing: 0.3px;

  &.status-instock {
    background: #c8e6c9;
    color: #256029;
  }

  &.status-outofstock {
    background: #ffcdd2;
    color: #c63737;
  }

  &.status-lowstock {
    background: #feedaf;
    color: #8a5340;
  }
}

.order-badge {
  border-radius: 2px;
  padding: 0.25em 0.5rem;
  text-transform: uppercase;
  font-weight: 700;
  font-size: 12px;
  letter-spacing: 0.3px;

  &.order-delivered {
    background: #c8e6c9;
    color: #256029;
  }

  &.order-cancelled {
    background: #ffcdd2;
    color: #c63737;
  }

  &.order-pending {
    background: #feedaf;
    color: #8a5340;
  }

  &.order-returned {
    background: #eccfff;
    color: #694382;
  }
}

@media screen and (max-width: 960px) {
  ::v-deep(.p-datatable) {
    &.p-datatable-customers {
      .p-datatable-thead > tr > th,
      .p-datatable-tfoot > tr > td {
        display: none !important;
      }

      .p-datatable-tbody > tr {
        border-bottom: 1px solid var(--surface-d);

        > td {
          text-align: left !important;
          display: block;
          border: 0 none !important;
          width: 100% !important;
          float: left;
          clear: left;
          border: 0 none;

          .p-column-title {
            padding: 0.4rem;
            min-width: 30%;
            display: inline-block;
            margin: -0.4rem 1rem -0.4rem -0.4rem;
            font-weight: bold;
          }

          .p-progressbar {
            margin-top: 0.5rem;
            display: inline-block;
            width: 60%;
          }

          .p-rating {
            display: inline-block;
          }
        }
      }

      .p-datatable-tbody > tr.p-rowgroup-footer {
        display: flex;
      }
    }
  }
}

.true-icon {
  color: #256029;
}

.false-icon {
  color: #c63737;
}
</style>



