**Android Recycleview**


# AsyncListDiffer

 AsyncListDiffer 相比较 DiffUtil.DiffResult 优势
1. 将 DiffUtil.calculateDiff() 比较方法放在了异步进行
2. 不需要自己处理复制新老数组，不需要手动刷新
3. 不需要额外发送更新数据，也不需要额外监听手动处理刷新

使用方法
1. 创建 DiffUtil.ItemCallback
2. 创建 AsyncListDiffer
3. 获取数据  AsyncListDiffer.getCurrentList()
4. 刷新数据 AsyncListDiffer.submitList(final List<T> newList, final Runnable commitCallback)

使用示例
```
class AdapterAsync : RecyclerView.Adapter<BaseRecyclerViewHolder> {
    val TAG = AdapterAsync::class.java.simpleName
    val DIFF_CALLBACK = object : DiffUtil.ItemCallback<ItemData>() {
        override fun areItemsTheSame(oldItem: ItemData, newItem: ItemData): Boolean {
            // User properties may have changed if reloaded from the DB, but ID is fixed
            return oldItem.title == newItem.title
        }

        override fun areContentsTheSame(oldItem: ItemData, newItem: ItemData): Boolean {
            // NOTE: if you use equals, your object must properly override Object#equals()
            // Incorrectly returning false here will result in too many animations.
            return oldItem == newItem;
        }
    }
    var mDiffer: AsyncListDiffer<ItemData>? = null

    constructor() : super() {
        mDiffer = AsyncListDiffer(this, DIFF_CALLBACK)
    }


    /**
     * 设置数据
     * @param list List<ItemData>
     */
    fun submitList(list: List<ItemData>?) {
        mDiffer?.submitList(list)
    }

    /**
     * 设置数据
     * @param list List<ItemData>?
     * @param commitCallback Runnable
     */
    fun submitList(list: List<ItemData>?, commitCallback: Runnable) {
        mDiffer?.submitList(list, commitCallback)
    }

    /**得到数据源
     *
     * @return MutableList<ItemData>?
     */
    fun getData(): List<ItemData>? {
        return mDiffer?.currentList
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): BaseRecyclerViewHolder {
        var itemView =
            LayoutInflater.from(parent.context).inflate(R.layout.layout_item_v1, parent, false)
        return BaseRecyclerViewHolder(itemView)
    }

    override fun getItemCount(): Int {
        return mDiffer?.currentList?.size ?: 0
    }

    override fun onBindViewHolder(holder: BaseRecyclerViewHolder, position: Int) {
        val mPosition = holder.bindingAdapterPosition
        val itemData = mDiffer?.currentList?.get(mPosition)
        Log.d(TAG, "刷新数据 mPosition：${mPosition}")
        itemData?.let {
            holder.get<TextView>(R.id.tv_title).text = "${it.id}  >> ${it.title}"
            holder.get<TextView>(R.id.tv_msg).text = "${it.msg}"
            if (it.bgColor != 0) {
                holder.itemView.setBackgroundColor(it.bgColor)
            }
        }
    }

    override fun onBindViewHolder(
        holder: BaseRecyclerViewHolder,
        position: Int,
        payloads: MutableList<Any>
    ) {
        super.onBindViewHolder(holder, position, payloads)
        Log.d(TAG, "刷新数据 payloads mPosition：${position}")
    }
}
```
