<template>
  <div :class="$_css.tableWrapper">
    <div class="vuetable-head-wrapper" v-if="isFixedHeader">
      <table :class="['vuetable fixed-header', $_css.tableClass, $_css.tableHeaderClass]">
        <vuetable-col-group :is-header="true"/>
        <thead>
          <slot name="tableHeader" :fields="tableFields">
            <template v-for="(header, headerIndex) in headerRows">
              <component :is="header" :key="headerIndex"
                @vuetable:header-event="onHeaderEvent"
              ></component>
            </template>
          </slot>
          <tr v-if="columnResize">
            <td v-for="(field, index) in visibleFields" class="resize-slider">
              <input type="range" min="1" max="100" value="100" style="width: 100%;"
                     class="resize-slider"
                     v-if="index < (visibleFields.length - 1)"
                     @mousedown="resizeCol($event, index)">
              <input type="range" min="1" max="100" value="100" style="width: 100%;"
                     class="resize-slider"
                     v-if="index === (visibleFields.length - 1)"
                     @click.prevent="noop" @mousedown.prevent="noop">
            </td>
          </tr>
        </thead>
      </table>
    </div>

    <div class="vuetable-body-wrapper" :class="{'fixed-header' : isFixedHeader}" :style="{height: tableHeight}">
      <table :class="['vuetable', isFixedHeader ? 'fixed-header' : '', $_css.tableClass, $_css.tableBodyClass]">
      <vuetable-col-group/>
      <thead v-if="!isFixedHeader">
        <slot name="tableHeader" :fields="tableFields">
          <template v-for="(header, headerIndex) in headerRows">
            <component :is="header" :key="headerIndex"
              @vuetable:header-event="onHeaderEvent"
            ></component>
          </template>
        </slot>
        <tr v-if="columnResize">
          <td v-for="(field, index) in visibleFields" class="resize-slider">
            <input type="range" min="1" max="100" value="100" style="width: 100%;"
                   class="resize-slider"
                   v-if="index < (visibleFields.length - 1)"
                   @mousedown="resizeCol($event, index)">
            <input type="range" min="1" max="100" value="100" style="width: 100%;"
                   class="resize-slider"
                   v-if="index === (visibleFields.length - 1)"
                   @click.prevent="noop" @mousedown.prevent="noop">
          </td>
        </tr>
      </thead>
      <tfoot>
        <slot name="tableFooter" :vuetable="vuetable" :fields="tableFields"></slot>
      </tfoot>
      <tbody v-cloak class="vuetable-body">
        <RecycleScroller :items="tableData" :min-item-size="54" class="scroller">
        <template slot-scope="{ item, index, active }">
          <tr :item-index="index"
              :key="index"
              :class="onRowClass(item, index)"
              @click="onRowClicked(item, index, $event)"
              @dblclick="onRowDoubleClicked(item, index, $event)"
              @mouseover="onMouseOver(item, index, $event)"
              @mouseout="onMouseOut(item, index, $event)"
              :draggable="allowDragdrop"
              @dragover="fireEvent('dragover', item, index, $event)"
              @dragenter="fireEvent('dragenter', item, index, $event)"
              @dragleave="fireEvent('dragleave', item, index, $event)"
              @dragstart="fireEvent('dragstart', item, index, $event)"
              @dragend="fireEvent('dragend', item, index, $event)"
              @drop="fireEvent('drop', item, index, $event)"
          >
            <template v-for="(field, fieldIndex) in tableFields">
              <template v-if="field.visible">
                <template v-if="isFieldComponent(field.name)">
                  <component :is="field.name"
                    :key="fieldIndex"
                    :row-data="item" :row-index="index" :row-field="field"
                    :vuetable="vuetable"
                    :class="bodyClass('vuetable-component', field)"
                    :style="{width: field.width}"
                    @vuetable:field-event="onFieldEvent"
                  ></component>
                </template>
                <template v-else-if="isFieldSlot(field.name)">
                  <td :class="bodyClass('vuetable-slot', field)"
                    :key="fieldIndex"
                    :style="{width: field.width}"
                  >
                    <slot :name="field.name"
                          :row-data="item" :row-index="index" :row-field="field"
                          :vuetable="vuetable"
                          @vuetable:field-event="onFieldEvent"
                    ></slot>
                  </td>
                </template>
                <template v-else>
                  <td :class="bodyClass('vuetable-td-'+field.name, field)"
                    :key="fieldIndex"
                    :style="{width: field.width}"
                    v-html="renderNormalField(field, item)"
                    @click="onCellClicked(item, index, field, $event)"
                    @dblclick="onCellDoubleClicked(item, index, field, $event)"
                    @contextmenu="onCellRightClicked(item, index, field, $event)"
                  ></td>
                </template>
              </template>
            </template>
          </tr>
          <template v-if="useDetailRow">
            <transition :name="detailRowTransition" :key="index">
              <tr v-if="isVisibleDetailRow(item[trackBy])"
                @click="onDetailRowClick(item, index, $event)"
                :class="onDetailRowClass(item, index)"
              >
                <td :colspan="countVisibleFields">
                  <component :is="detailRowComponent"
                    :row-data="item"
                    :row-index="index"
                    :options="detailRowOptions"
                  ></component>
                </td>
              </tr>
            </transition>
          </template>
        </template>
        </RecycleScroller>
        <template v-if="displayEmptyDataRow">
          <slot name="noDataTemplate" :vuetable="vuetable">
            <tr>
              <td :colspan="countVisibleFields"
                class="vuetable-empty-result"
                v-html="noDataTemplate"
              ></td>
            </tr>
          </slot>
        </template>
        <template v-if="lessThanMinRows">
          <tr v-for="i in blankRows" class="blank-row" :key="i">
            <template v-for="(field, fieldIndex) in tableFields">
              <td v-if="field.visible" :key="fieldIndex">&nbsp;</td>
            </template>
          </tr>
        </template>
      </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import VuetableRowHeader from './VuetableRowHeader'
import VuetableColGroup from './VuetableColGroup'
import RecycleScroller from './RecycleScroller'
import CssSemanticUI from './VuetableCssSemanticUI.js'
import InfiniteScrollMixin from './VuetableInfiniteScrollMixin'

export default {
  name: 'Vuetable',
  mixins: [InfiniteScrollMixin],
  components: {
    VuetableRowHeader,
    VuetableColGroup,
    RecycleScroller
  },

  props: {
    allowDragdrop: {
      type: Boolean,
      default: false,
    },
    fields: {
      type: Array,
      required: true
    },
    loadOnStart: {
      type: Boolean,
      default: true
    },
    apiUrl: {
      type: String,
      default: ''
    },
    httpMethod: {
      type: String,
      default: 'get',
      validator: (value) => {
        return ['get', 'post'].indexOf(value) > -1
      }
    },
    reactiveApiUrl: {
      type: Boolean,
      default: true
    },
    apiMode: {
      type: Boolean,
      default: true
    },
    data: {
      type: [Array, Object],
      default: null
    },
    dataManager: {
      type: Function,
      default: null
    },
    dataPath: {
      type: String,
      default: 'data'
    },
    paginationPath: {
      type: String,
      default: 'links.pagination'
    },
    queryParams: {
      type: [Object, Function],
      default () {
        return {
          sort: 'sort',
          page: 'page',
          perPage: 'per_page'
        }
      }
    },
    appendParams: {
      type: Object,
      default () {
        return {}
      }
    },
    httpOptions: {
      type: Object,
      default () {
        return {}
      }
    },
    httpFetch: {
      type: Function,
      default: null
    },
    perPage: {
        type: Number,
        default: 10
    },
    /**
     * Page that should be displayed when the table is first displayed
     */
    initialPage: {
      type: Number,
      default: 1
    },
    /**
     * First page number. Set this prop to 0 for zero based pagination
     */
    firstPage: {
      type: Number,
      default: 1
    },
    sortOrder: {
      type: Array,
      default () {
        return []
      }
    },
    multiSort: {
      type: Boolean,
      default: false
    },
    tableHeight: {
      type: String,
      default: null
    },
    /*
     * physical key that will trigger multi-sort option
     * possible values: 'alt', 'ctrl', 'meta', 'shift'
     * 'ctrl' might not work as expected on Mac
     */
    multiSortKey: {
      type: String,
      default: 'alt'
    },
    rowClass: {
      type: [String, Function],
      default: ''
    },
    detailRowComponent: {
      type: [String, Object],
      default: ''
    },
    detailRowTransition: {
      type: String,
      default: ''
    },
    detailRowClass: {
      type: [String, Function],
      default: 'vuetable-detail-row'
    },
    detailRowOptions: {
      type: Object,
      default() {
        return {}
      }
    },
    trackBy: {
      type: String,
      default: 'id'
    },
    css: {
      type: Object,
      default () {
        return {}
      }
    },
    minRows: {
      type: Number,
      default: 0
    },
    silent: {
      type: Boolean,
      default: false
    },
    noDataTemplate: {
      type: String,
      default() {
        return 'No Data Available'
      }
    },
    showSortIcons: {
      type: Boolean,
      default: true
    },
    headerRows: {
      type: Array,
      default() {
        return ['VuetableRowHeader']
      }
    },
    transform: {
      type: Function,
      default: null
    },
    sortParams: {
      type: Function,
      default: null
    },
    fieldPrefix: {
      type: String,
      default() {
        return 'vuetable-field-'
      }
    },
    eventPrefix: {
      type: String,
      default() {
        return 'vuetable:'
      }
    },
    columnResize: {
      type: Boolean,
      default: false,
    }
  },

  data () {
    return {
      tableFields: [],
      tableData: [{
        id: 1,
        name: 'sagar'
      },{
        id: 2,
        name: 'goyal'
      }],
      tablePagination: null,
      currentPage: this.initialPage,
      selectedTo: [],
      visibleDetailRows: [],
      lastScrollPosition: 0,
      scrollBarWidth: '17px', //chrome default
      scrollVisible: false,
      $_css: {}
    }
  },

  computed: {
    version: () => VERSION,
    useDetailRow () {
      if ( ! this.dataIsAvailable) return false

      return this.detailRowComponent !== ''
    },
    dataIsAvailable () {
      if ( ! this.tableData) return false
      return this.tableData.length > 0
    },
    hasRowIdentifier () {
      return this.tableData && typeof(this.tableData[0][this.trackBy]) !== 'undefined'
    },
    visibleFields() {
      return this.tableFields.filter( (field) => {
        return field.visible
      })
    },
    countVisibleFields () {
      return this.visibleFields.length
    },
    countTableData () {
      if (this.tableData === null) {
        return 0
      }
      return this.tableData.length
    },
    displayEmptyDataRow () {
      return this.countTableData === 0 && this.noDataTemplate.length > 0
    },
    lessThanMinRows () {
      if (this.tableData === null || this.tableData.length === 0) {
        return true
      }
      return this.tableData.length < this.minRows
    },
    blankRows () {
      if (this.tableData === null || this.tableData.length === 0) {
        return this.minRows
      }
      if (this.tableData.length >= this.minRows) {
        return 0
      }

      return this.minRows - this.tableData.length
    },
    isApiMode () {
      return this.apiMode
    },
    isDataMode () {
      return ! this.apiMode
    },
    isFixedHeader () {
      return this.tableHeight != null
    },
    vuetable () {
      return this
    }
  },

  created() {
    this.mergeCss()
    this.normalizeFields()
    this.normalizeSortOrder()
    this.$nextTick( () => {
      this.fireEvent('initialized', this.tableFields)
    })
  },

  mounted () {
    if (this.loadOnStart) {
      this.loadData()
    }

    if (this.isFixedHeader) {
      this.scrollBarWidth = this.getScrollBarWidth() + 'px';

      let elem = this.$el.getElementsByClassName('vuetable-body-wrapper')[0];
      if (elem != null) {
        elem.addEventListener('scroll', this.handleScroll);
      }
    }
  },

  destroyed () {
    let elem = this.$el.getElementsByClassName('vuetable-body-wrapper')[0];
    if (elem != null) {
      elem.removeEventListener('scroll', this.handleScroll);
    }
  },

  watch: {
    multiSort (newVal, oldVal) {
      if (newVal === false && this.sortOrder.length > 1) {
        this.sortOrder.splice(1);
        this.loadData();
      }
    },

    apiUrl (newVal, oldVal) {
      if (this.reactiveApiUrl && newVal !== oldVal)
        this.refresh()
    },

    data (newVal, oldVal) {
      this.setData(newVal)
    },

    tableHeight (newVal, oldVal) {
      this.checkScrollbarVisibility()
    },

    fields (newVal, oldVal) {
    	this.normalizeFields();
    },

    perPage (newVal, oldVal) {
      this.reload();
    }
},

  methods: {
    resizeCol (event, index) {
      event.preventDefault();
      const rightHead = index < this.countVisibleFields - 1
        ? this.visibleFields[index + 1] : null;
      const currHead = this.visibleFields[index];
      const units = currHead.width.replace(/[0-9.]+/g, '');
      const oldWidth = parseFloat(currHead.width.replace(units, ''));
      const widthInPx = window.getComputedStyle(event.target).getPropertyValue('width');
      const unitsToPx = parseFloat(widthInPx.replace('px', '')) / oldWidth;
      const startX = event.pageX;
      const onMouseMove = function _onMouseMove(e) {
        e.preventDefault();
        const delta = e.pageX - startX;
        if(Math.abs(delta) < 5) return;
        const deltaInOldUnits = delta / unitsToPx;
        const newWidth = (oldWidth + deltaInOldUnits).toFixed(2);
        if(newWidth < (currHead.minWidth || 5)) return;
        currHead.width = `${(newWidth)}${units}`;
        if(units === 'px' && rightHead) {
            rightHead.width = `${(rightHead.width - deltaInOldUnits)}${units}`;
        }
      };
      const onMouseUp = () => {
        document.removeEventListener('mousemove', onMouseMove);
        document.removeEventListener('mouseup', onMouseUp);
        const rightHeadWidth = rightHead ? rightHead.width : null;
        this.fireEvent('col-resize', index, currHead.width, rightHeadWidth);
      };
      document.addEventListener('mousemove', onMouseMove);
      document.addEventListener('mouseup', onMouseUp);
    },
    getScrollBarWidth () {
      const outer = document.createElement('div');
      const inner = document.createElement('div');

      outer.style.visibility = 'hidden';
      outer.style.width = '100px';

      inner.style.width = '100%';

      outer.appendChild(inner);
      document.body.appendChild(outer);

      const widthWithoutScrollbar = outer.offsetWidth;
      outer.style.overflow = 'scroll';
      const widthWithScrollbar = inner.offsetWidth;
      document.body.removeChild(outer);

      return (widthWithoutScrollbar - widthWithScrollbar);
    },

    //make sure that the header and the body are aligned when scrolling horizontally on a table that is wider than the viewport
    handleScroll (e) {
      let horizontal = e.currentTarget.scrollLeft;

      //don't modify header scroll if we are scrolling vertically
      if (horizontal != this.lastScrollPosition) {
        let header = this.$el.getElementsByClassName('vuetable-head-wrapper')[0]
        if (header != null) {
          header.scrollLeft = horizontal;
        }
        this.lastScrollPosition = horizontal;
      }
    },

    mergeCss () {
      this.$_css = { ...CssSemanticUI.table, ...this.css }
    },

    bodyClass (base, field) {
      return [ base, field.dataClass ]
    },

    normalizeFields () {
      if (typeof(this.fields) === 'undefined') {
        this.warn('You need to provide "fields" prop.')
        return
      }

      this.tableFields = []

      this.fields.forEach( (field, i) => {
        this.tableFields.push(this.newField(field, i))
      })
    },

    newField (field, index) {
      let defaultField = {
        name: '',
        // title:
        // this allow the code to detect undefined title
        // and replace it with capitalized name instead
        titleClass: '',
        dataClass: '',
        sortField: null,
        formatter: null,
        visible: true,
        width: null,
        $_index: index,
      }

      if (typeof(field) === 'string') {
        return Object.assign({}, defaultField, {
          name: this.normalizeFieldName(field),
          title: this.makeTitle(field),
        })
      }

      let obj = Object.assign({}, defaultField, field)
      obj.name = this.normalizeFieldName(obj.name)
      if (obj.title === undefined) {
        obj.title = this.makeTitle(obj.name)
      }
      if (obj.formatter !== null && typeof(obj.formatter) !== 'function') {
        console.error(obj.name + ' field formatter must be a function')
        obj.formatter = null
      }
      return obj
    },

    normalizeFieldName (fieldName) {
      if (fieldName instanceof Object) return fieldName

      return typeof(fieldName) === 'string' && fieldName.replace('__', this.fieldPrefix)
    },

    setData (data) {
      if (data === null || typeof(data) === 'undefined') return

      this.fireEvent('loading')

      if (Array.isArray(data)) {
        this.tableData = data
        console.log('in created function items', this.tableData)
        this.fireEvent('loaded')
        return
      }

      this.tableData = this.getObjectValue(data, this.dataPath, null)
      this.tablePagination = this.getObjectValue(data, this.paginationPath, null)

      this.$nextTick( () => {
        this.checkIfRowIdentifierExists()
        this.updateHeader()
        this.fireEvent('pagination-data', this.tablePagination)
        this.fireEvent('loaded')
      })
    },

    checkIfRowIdentifierExists () {
      if (! this.dataIsAvailable) return

      if ( ! this.hasRowIdentifier) {
        this.warn('Invalid your data! Use "track-by" prop to specify.')
        return false
      }

      return true
    },

    makeTitle (str) {
      if (this.isFieldComponent(str)) {
        return ''
      }

      return this.titleCase(str.replace('.', ' '))
    },

    getFieldTitle (field) {
      if (typeof(field.title) === 'function') return field.title()

      return field.title
    },

    renderNormalField (field, item) {
      return this.hasFormatter(field)
        ? this.callFormatter(field, item)
        : this.getObjectValue(item, field.name, '')
    },

    isFieldComponent (fieldName) {
      if (fieldName instanceof Object) {
        // let's assume it is a Vue component
        return true
      }

      return fieldName.slice(0, this.fieldPrefix.length) === this.fieldPrefix
        || fieldName.slice(0, 2) === '__'
    },

    isFieldSlot (fieldName) {
      return typeof this.$scopedSlots[fieldName] !== 'undefined'
    },

    titleCase (str) {
      return str.replace(/\w+/g, (txt) => {
        return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase()
      })
    },

    camelCase (str, delimiter = '_') {
      return str.split(delimiter).map( (item) => self.titleCase(item) ).join('')
    },

    loadData (success = this.loadSuccess, failed = this.loadFailed) {
      if (this.isDataMode) {
        this.handleDataMode()
        return
      }

      this.fireEvent('loading')

      this.httpOptions['params'] = this.getAppendParams( this.getAllQueryParams() )

      return this.fetch(this.apiUrl, this.httpOptions).then(
          success,
          failed
      ).catch(() => failed())
    },

    fetch (apiUrl, httpOptions) {
      if (this.httpFetch) {
        return this.httpFetch(apiUrl, httpOptions)
      }

      if (this.httpMethod === 'get') {
        return axios.get(apiUrl, httpOptions)
      }
      else { // Is a POST request
        let params = httpOptions.params
        delete httpOptions.params
        return axios.post(apiUrl, params, httpOptions)
      }
    },

    loadSuccess (response) {
      this.fireEvent('load-success', response)

      let body = this.transform ? this.transform(response.data) : response.data

      let newData = this.getObjectValue(body, this.dataPath, null)
      this.tableData = this.infiniteScroll ? this.tableData.concat(newData) : newData
      this.tablePagination = this.getObjectValue(body, this.paginationPath, null)

      if (this.tablePagination === null) {
        this.warn('vuetable: pagination-path "' + this.paginationPath + '" not found. '
          + 'It looks like the data returned from the server does not have pagination information '
          + "or you may have set it incorrectly.\n"
          + 'You can explicitly suppress this warning by setting pagination-path="".'
        )
      }

      this.$nextTick( () => {
        this.checkIfRowIdentifierExists()
        this.updateHeader()
        this.fireEvent('pagination-data', this.tablePagination)
        this.fireEvent('loaded')
      })
    },

    updateHeader () {
      // $nextTick doesn't seem to work in all cases. This might be because
      // $nextTick is finished before the transition element (just my guess)
      //
      // the scrollHeight value does not yet changed, causing scrollVisible
      // to remain "true", therefore, the header gutter never gets updated
      // to reflect the display of scrollbar in the table body.
      // setTimeout 80ms seems to work in this case.
      setTimeout(this.checkScrollbarVisibility, 80)
    },

    checkScrollbarVisibility () {
      this.$nextTick( () => {
        let elem = this.$el.getElementsByClassName('vuetable-body-wrapper')[0]
        if (elem != null) {
          this.scrollVisible = (elem.scrollHeight > elem.clientHeight)
          this.fireEvent('scrollbar-visible', this.scrollVisible)
        }
      })
    },

    loadFailed (response) {
      console.error('load-error', response)
      this.fireEvent('load-error', response)
      this.fireEvent('loaded')
    },

    fireEvent () {
      if (arguments.length === 1) {
        return this.$emit(this.eventPrefix + arguments[0])
      }

      if (arguments.length > 1) {
        let args = Array.from(arguments)
        args[0] = this.eventPrefix + args[0]
        return this.$emit.apply(this, args)
      }
    },

    warn (msg) {
      if (!this.silent) {
        console.warn(msg)
      }
    },

    getAllQueryParams () {
      let params = {}

      if (typeof(this.queryParams) === 'function') {
        params = this.queryParams(this.sortOrder, this.currentPage, this.perPage)
        return typeof(params) === 'object' ? params : {}
      }

      params[this.queryParams.sort] = this.getSortParam()
      params[this.queryParams.page] = this.currentPage
      params[this.queryParams.perPage] = this.perPage

      return params
    },

    getSortParam () {
      if (!this.sortOrder || this.sortOrder.field == '') {
        return ''
      }

      if (typeof this.sortParams === 'function') {
        return this.sortParams(this.sortOrder)
      }

      return this.getDefaultSortParam()
    },

    getDefaultSortParam () {
      return this.sortOrder.map( (item) => `${item.sortField}|${item.direction}`).join(',')
    },

    getAppendParams (params) {
      for (let x in this.appendParams) {
        params[x] = this.appendParams[x]
      }

      return params
    },

    isSortable (field) {
      return field.sortField !== null
    },

    currentSortOrderPosition (field) {
      if ( ! this.isSortable(field)) {
        return false
      }

      for (let i = 0; i < this.sortOrder.length; i++) {
        if (this.fieldIsInSortOrderPosition(field, i)) {
          return i;
        }
      }

      return false;
    },

    fieldIsInSortOrderPosition (field, i) {
      return this.sortOrder[i].sortField === field.sortField
    },

    orderBy (field, event) {
      if ( ! this.isSortable(field) ) return

      let key = this.multiSortKey.toLowerCase() + 'Key'

      if (this.multiSort && event[key]) { //adding column to multisort
        this.multiColumnSort(field)
      } else {
        //no multisort, or resetting sort
        this.singleColumnSort(field)
      }

      this.currentPage = this.firstPage    // reset page index
      if (this.apiMode || this.dataManager) {
        this.loadData()
      }
    },

    addSortColumn (field, direction) {
      this.sortOrder.push({
        field: field.name,
        sortField: field.sortField,
        direction: 'asc'
      });
    },

    removeSortColumn (index) {
      this.sortOrder.splice(index, 1);
    },

    setSortColumnDirection (index, direction) {
      this.sortOrder[index].direction = direction
    },

    multiColumnSort (field) {
      let i = this.currentSortOrderPosition(field);

      if (i === false) { //this field is not in the sort array yet
        this.addSortColumn(field, 'asc')
      } else { //this field is in the sort array, now we change its state
        if (this.sortOrder[i].direction === 'asc') {
          // switch direction
          this.setSortColumnDirection(i, 'desc')
        } else {
          this.removeSortColumn(i)
        }
      }
    },

    singleColumnSort (field) {
      if (this.sortOrder.length === 0) {
        // this.clearSortOrder()
        this.addSortColumn(field, 'asc')
        return
      }

      this.sortOrder.splice(1); //removes additional columns

      if (this.fieldIsInSortOrderPosition(field, 0)) {
        // change sort direction
        this.sortOrder[0].direction = this.sortOrder[0].direction === 'asc' ? 'desc' : 'asc'
      } else {
        // reset sort direction
        this.sortOrder[0].direction = 'asc'
      }
      this.sortOrder[0].field = field.name
      this.sortOrder[0].sortField = field.sortField
    },

    clearSortOrder () {
      this.sortOrder = []
    },

    hasFormatter (item) {
      return typeof(item.formatter) === 'function'
    },

    callFormatter (field, item) {
      if ( ! this.hasFormatter(field)) return

      if (typeof(field.formatter) === 'function') {
       return field.formatter(this.getObjectValue(item, field.name), this)
      }
    },

    getObjectValue (object, path, defaultValue) {
      defaultValue = (typeof defaultValue === 'undefined') ? null : defaultValue

      let obj = object
      if (path.trim() != '') {
        let keys = path.split('.')
        keys.forEach( (key) => {
          if (obj !== null && typeof obj[key] !== 'undefined' && obj[key] !== null) {
            obj = obj[key]
          } else {
            obj = defaultValue
            return
          }
        })
      }
      return obj
    },

    selectId (key) {
      if ( ! this.isSelectedRow(key)) {
        this.selectedTo.push(key)
      }
    },

    unselectId (key) {
      this.selectedTo = this.selectedTo.filter( (item) => {
        return item !== key
      })
    },

    isSelectedRow (key) {
      return this.selectedTo.indexOf(key) >= 0
    },

    clearSelectedValues () {
      this.selectedTo = []
    },

    gotoPreviousPage () {
      if (this.currentPage > this.firstPage) {
        this.currentPage--
        this.loadData()
      }
    },

    gotoNextPage () {
      if (this.currentPage < this.tablePagination.last_page) {
        this.currentPage++
        this.loadData()
      }
    },

    gotoPage (page) {
      if (page != this.currentPage && (page >= this.firstPage && page <= this.tablePagination.last_page)) {
        this.currentPage = page
        this.loadData()
      }
    },

    isVisibleDetailRow (rowId) {
      return this.visibleDetailRows.indexOf( rowId ) >= 0
    },

    showDetailRow (rowId) {
      if (!this.isVisibleDetailRow(rowId)) {
        this.visibleDetailRows.push(rowId)
      }
      this.checkScrollbarVisibility()
    },

    hideDetailRow (rowId) {
      if (this.isVisibleDetailRow(rowId)) {
        this.visibleDetailRows.splice(
          this.visibleDetailRows.indexOf(rowId),
          1
        )
        this.updateHeader()
      }
    },

    toggleDetailRow (rowId) {
      if (this.isVisibleDetailRow(rowId)) {
        this.hideDetailRow(rowId)
      } else {
        this.showDetailRow(rowId)
      }
    },

    showField (index) {
      if (index < 0 || index > this.tableFields.length) return

      this.tableFields[index].visible = true
    },

    hideField (index) {
      if (index < 0 || index > this.tableFields.length) return

      this.tableFields[index].visible = false
    },

    toggleField (index) {
      if (index < 0 || index > this.tableFields.length) return

      this.tableFields[index].visible = ! this.tableFields[index].visible
    },

    makePagination (total = null, perPage = null, currentPage = null) {
      let pagination = {}
      total = total === null ? 0 : total
      perPage = perPage === null ? this.perPage : perPage
      currentPage = currentPage === null ? this.currentPage : currentPage

      return {
        'total': total,
        'per_page': perPage,
        'current_page': currentPage,
        'last_page': Math.ceil(total / perPage) || 0,
        'next_page_url': '',
        'prev_page_url': '',
        'from': (currentPage -1) * perPage +1,
        'to': Math.min(currentPage * perPage, total)
      }
    },

    normalizeSortOrder () {
      this.sortOrder.forEach( (item) => {
        item.sortField = item.sortField || item.field
      })
    },

    handleDataMode () {
      // data is array
      if (this.data !== null && Array.isArray(this.data)) {
        this.setData(this.data)
        return
      }

      // data must be an object, check if dataManager is present
      if (this.dataManager) {
        this.callDataManager()
      } else {
        this.setData(this.data)
      }
    },

    callDataManager () {
      const result = this.dataManager(this.sortOrder, this.makePagination())

      if (this.isPromiseObject(result)) {
        result.then(data => this.setData(data))
      } else {
        this.setData(result)
      }
    },

    isObject (unknown) {
      return typeof(unknown) === "object" && unknown !== null
    },

    isPromiseObject (unknown) {
      return this.isObject(unknown) && typeof(unknown.then) === "function"
    },

    onRowClass (dataItem, index) {
      if (typeof(this.rowClass) === 'function') {
        return this.rowClass(dataItem, index)
      }

      return this.rowClass
    },

    onDetailRowClass (dataItem, index) {
      if (typeof(this.detailRowClass) === 'function') {
        return this.detailRowClass(dataItem, index)
      }

      return this.detailRowClass
    },

    onRowClicked (dataItem, dataIndex, event) {
      this.fireEvent('row-clicked', { data: dataItem, index: dataIndex, event: event })
      return true
    },

    onRowDoubleClicked (dataItem, dataIndex, event) {
      this.fireEvent('row-dblclicked', { data: dataItem, index: dataIndex, event: event })
    },

    onDetailRowClick (dataItem, dataIndex, event) {
      this.fireEvent('detail-row-clicked', { data: dataItem, index: dataIndex, event: event })
    },

    onCellClicked (dataItem, dataIndex, field, event) {
      this.fireEvent('cell-clicked', { data: dataItem, index: dataIndex, field: field, event: event })
    },

    onCellDoubleClicked (dataItem, dataIndex, field, event) {
      this.fireEvent('cell-dblclicked', { data: dataItem, index: dataIndex, field: field, event: event })
    },

    onCellRightClicked (dataItem, dataIndex, field, event) {
      this.fireEvent('cell-rightclicked', { data: dataItem, index: dataIndex, field: field, event: event })
    },

    onMouseOver (dataItem, dataIndex, event) {
      this.fireEvent('row-mouseover', { data: dataItem, index: dataIndex, event: event})
    },

    onMouseOut (dataItem, dataIndex, event) {
      this.fireEvent('row-mouseout', { data: dataItem, index: dataIndex, event: event})
    },

    onFieldEvent (type, payload) {
      this.fireEvent('field-event', type, payload, this)
    },

    onHeaderEvent (type, payload) {
      this.fireEvent('header-event', type, payload, this)
    },

    onCheckboxToggled (isChecked, fieldName, dataItem) {
      let idColumn = this.trackBy

      if (dataItem[idColumn] === undefined) {
        this.warn('checkbox field: The "'+this.trackBy+'" field does not exist! Make sure the field you specify in "track-by" prop does exist.')
        return
      }

      let key = dataItem[idColumn]
      if (isChecked) {
        this.selectId(key)
      } else {
        this.unselectId(key)
      }

      this.fireEvent('checkbox-toggled', isChecked, fieldName, dataItem)
    },

    onCheckboxToggledAll (isChecked) {
      let idColumn = this.trackBy

      if (isChecked) {
        this.tableData.forEach( (dataItem) => {
          this.selectId(dataItem[idColumn])
        })
      } else {
        this.tableData.forEach( (dataItem) => {
          this.unselectId(dataItem[idColumn])
        })
      }

      this.fireEvent('checkbox-toggled-all', isChecked)
    },

    /*
     * API for externals
     */
    changePage (page) {
      if (page === 'prev') {
        this.gotoPreviousPage()
      } else if (page === 'next') {
        this.gotoNextPage()
      } else {
        this.gotoPage(page)
      }
    },

    reload () {
      return this.loadData()
    },

    refresh () {
      this.currentPage = this.firstPage
      return this.loadData()
    },

    resetData () {
      this.tableData = null
      this.tablePagination = null
      this.fireEvent('data-reset')
    },

    noop() {

    }
  }, // end: methods
}
</script>

<style>
  [v-cloak] {
    display: none;
  }
  table.vuetable.fixed-header {
    table-layout: fixed;
  }
  .vuetable th.sortable:hover {
    color: #2185d0;
    cursor: pointer;
  }
  .vuetable-head-wrapper {
    overflow-x: hidden;
  }
  .vuetable-head-wrapper table.vuetable {
    border-bottom-left-radius: 0px;
    border-bottom-right-radius: 0px;
  }
  .vuetable-body-wrapper.fixed-header {
    position:relative;
    overflow-y:auto;
  }
  .vuetable-body-wrapper table.vuetable.fixed-header {
    border-top:none !important;
    margin-top:0 !important;
    border-top-left-radius: 0px;
    border-top-right-radius: 0px;
  }
  .vuetable-empty-result {
    text-align: center;
  }

  input.resize-slider {
    -webkit-appearance: none;
    width: 100%;
    margin: 5px 0;
  }

  input.resize-slider::-webkit-slider-runnable-track {
    width: 100%;
    height: 5px;
    background: #999999;
  }
  input.resize-slider::-webkit-slider-thumb {
    box-shadow: 1px 1px 1px #000000, 0px 0px 1px #0d0d0d;
    height: 15px;
    width: 4px;
    border-radius: 1px;
    background: #000000;
    cursor: ew-resize;
    -webkit-appearance: none;
    margin-top: -5px;
  }
  input.resize-slider:focus::-webkit-slider-runnable-track {
    background: #ebebeb;
  }
  input.resize-slider::-moz-range-track {
    width: 100%;
    height: 5px;
    box-shadow: 0px 0px 0px #000000, 0px 0px 0px #0d0d0d;
    background: #999999;
  }
  input.resize-slider::-moz-range-thumb {
    box-shadow: 1px 1px 1px #000000, 0px 0px 1px #0d0d0d;
    border: 0px solid #000000;
    height: 15px;
    width: 4px;
    border-radius: 1px;
    background: #000000;
    cursor: ew-resize;
  }
  input.resize-slider::-ms-track {
    width: 100%;
    height: 5px;
    background: transparent;
    border-color: transparent;
    color: transparent;
  }
  input.resize-slider::-ms-fill-lower {
    background: #474747;
    border: 0px solid #010101;
    border-radius: 4px;
    box-shadow: 0px 0px 0px #000000, 0px 0px 0px #0d0d0d;
  }
  input.resize-slider::-ms-fill-upper {
    background: #999999;
    border: 0px solid #010101;
    border-radius: 4px;
    box-shadow: 0px 0px 0px #000000, 0px 0px 0px #0d0d0d;
  }
  input.resize-slider::-ms-thumb {
    box-shadow: 1px 1px 1px #000000, 0px 0px 1px #0d0d0d;
    border: 0px solid #000000;
    height: 15px;
    width: 4px;
    border-radius: 1px;
    background: #000000;
    cursor: ew-resize;
  }
  input.resize-slider:focus::-ms-fill-lower {
    background: #999999;
  }
  input.resize-slider:focus::-ms-fill-upper {
    background: #ebebeb;
  }
  td.resize-slider {
    padding: 4px 0!important;
  }
</style>
