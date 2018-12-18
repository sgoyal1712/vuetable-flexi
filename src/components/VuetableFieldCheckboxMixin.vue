<script>
import VuetableFieldMixin from './VuetableFieldMixin.vue'

export default {
  mixins: [VuetableFieldMixin],
  data() {
    return {
      isHeaderIndeterminate: false,
    }
  },
  methods: {
    toggleCheckbox(dataItem, event) {
      this.vuetable.onCheckboxToggled(event.target.checked, this.rowField.name, dataItem)
    },

    toggleAllCheckbox(event) {
      this.vuetable.onCheckboxToggledAll(event.target.checked)
    },

    isSelected(rowData) {
      return this.vuetable.isSelectedRow(rowData[this.vuetable.trackBy])
    },

    isAllItemsInCurrentPageSelected() {
      if (! this.vuetable.tableData) return

      let idColumn = this.vuetable.trackBy
      let selected = this.vuetable.tableData.filter( (item) => this.vuetable.isSelectedRow(item[idColumn]) )

      // count == 0, clear the checkbox
      if (selected.length <= 0) {
        this.isHeaderIndeterminate = false
        return false
      }
      // count > 0 and count < perPage, set checkbox state to 'indeterminate'
      else if (selected.length < this.vuetable.perPage) {
        this.isHeaderIndeterminate = true
        return false
      }
      // count == perPage, set checkbox state to 'checked'
      else {
        this.isHeaderIndeterminate = false
        return true
      }
    }
  }
}
</script>
