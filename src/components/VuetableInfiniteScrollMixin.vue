<script>
  export default {
    data() {
      return {
        isLoadingData: false,
        rowElemHeight: 0,
        lastClientHeight: 0,
        lastEventTarget: null,
        tBody: null,
      };
    },
    props: {
      infiniteScroll: {
        type: Boolean,
        default: false,
      },
      numRowsToBottom: {
        type: Number,
        default: 5,
      },
    },
    methods: {
      handleInfiniteScroll({ target }) {
        const { clientHeight, scrollTop, scrollHeight } = target;
        if (this.isLoadingData || this.hasNoMoreData) return;
        if (clientHeight + scrollTop > scrollHeight - (this.numRowsToBottom * this.rowElemHeight)) {
          this.lastClientHeight = scrollTop;
          this.lastEventTarget = target;
          this.gotoNextPage();
        }
      },
      dataReceived() {
        if (this.lastEventTarget) { this.lastEventTarget.scrollTop = this.lastClientHeight - 50; }
        this.isLoadingData = false;
        this.lastEventTarget = null;
        // the below call is required for the case where the first
        // page is insufficient to fill the whole table. No scrollbar is rendered
        // and hence the scroll event is never fired
        this.$nextTick(() => {
          this.handleInfiniteScroll({ target: this.tBody });
        });
      },
      makingDataCall() {
        this.isLoadingData = true;
      },
    },
    computed: {
      hasNoMoreData() {
        if (!this.tablePagination) return true;
        return this.tableData.length === this.tablePagination.total;
      },
    },
    mounted() {
      if (!this.isFixedHeader || !this.infiniteScroll) return;
      const rowElem = this.$el.querySelectorAll('tbody.vuetable-body tr')[0];
      [this.tBody] = this.$el.getElementsByClassName('vuetable-body-wrapper');
      if (!this.tBody || !rowElem) return;
      this.rowElemHeight = rowElem.clientHeight;
      this.tBody.addEventListener('scroll', this.handleInfiniteScroll);
      this.$on('vuetable:loaded', this.dataReceived);
      this.$on('vuetable:loading', this.makingDataCall);
    },
    destroyed() {
      if (this.tBody != null) {
        this.tBody.removeEventListener('scroll', this.handleInfiniteScroll);
        this.$off('vuetable:loaded', this.dataReceived);
        this.$off('vuetable:loading', this.makingDataCall);
      }
    },
  };
</script>
