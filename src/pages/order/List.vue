<template>
    <div>
        <!-- 按钮 -->
        <el-button type="success" size="small" @click="toAddHandler">添加</el-button>
        <el-button type="danger" size="small">批量删除</el-button>
        <!-- 表格 -->
        <el-table :data="order.list">
         <el-table-column prop="id" label="编号"></el-table-column>
         <el-table-column prop="orderTime" label="下单时间"></el-table-column>         
         <el-table-column prop="total" label="总价"></el-table-column>
         <el-table-column prop="status" label="状态"></el-table-column>        
         <el-table-column prop="customerId" label="顾客id"></el-table-column>
         <el-table-column prop="waiterId" label="员工id"></el-table-column>
         <el-table-column prop="addressId" label="地址"></el-table-column>
         <el-table-column label="操作">
             <template v-slot="slot">
                 <a href="" @click.prevent="toDeleteHandler(slot.row.id)">删除</a>
                 <a href="" @click.prevent="toUpdateHandler(slot.row)">修改</a>
             </template>
         </el-table-column>
        </el-table>
        <!-- 分页 -->
        <el-pagination layout="prev, pager, next" :total="order.total" @current-change="pageChangeHandler">           
        </el-pagination>
        <!-- 模态框 -->
        <el-dialog  :title="title" :visible.sync="visible" width="60%">
            {{form}}
            <el-form :model="form" labei-width="80px"> 
               <el-form-item label="下单时间">       
                 <el-input v-model="form.orderTime"></el-input>
               </el-form-item>
               <el-form-item label="总价">
                   <el-input v-model="form.total"></el-input>
                </el-form-item>
                <el-form-item label="性别">
                   <el-input v-model="form.gender"></el-input>
                </el-form-item>
                <el-form-item label="真实姓名">
                   <el-input v-model="form.realname"></el-input>
                </el-form-item>
                <el-form-item label="手机号">
                   <el-input v-model="form.telephone"></el-input>
                </el-form-item>                 
             </el-form>

            <span></span>

            <span slot="footer" class="dialog-footer">
                <el-button size="samll" @click="closeModalHandler">取 消</el-button>
                <el-button size="samll" type="primary" @click="submitHandler">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>


<script>
import request from '@/utils/request'
import querystring from 'querystring'
export default {
    // 用于存放网页中需要调用的方法
    methods:{
        pageChangeHandler(page){
            // 将params中当前页改为插件中的当前页
           this.params.page=page-1;
           this.loadData();

        },

        loadData(){
           let url="http://localhost:6677/order/queryPage"
            request({
              url,
              method:"post",
              headers:{
                 "Content-Type":"application/x-www-form-urlencoded"
              },
              data:querystring.stringify(this.params)
            }).then((response)=>{
                this.order=response.data;
            })
        },
        
        submitHandler(){
            // this.form 对象--字符串-->后台
            // 通过request与后台交互，并且携带参数
            let url="http://localhost:6677/order/saveOrUpdate";
            request({
                url,
                method:"POST",
                headers:{
                    "Content-Type":"application/x-www-form-urlencoded"
                },
                data:querystring.stringify(this.form)

            }).then((response)=>{
                // 模态框关闭
                this.closeModalHandler();

                this.loadData();
                // 提示消息
                this.$message({
                    type:"success",
                    message:response.message

                })

            })
        },
        toDeleteHandler(id){
        //  确认
           this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
           }).then(() => {
                //   调用后台接口完成删除
                let url="http://localhost:6677/order/deleteById?id="+id;
                request.get(url).then((response)=>{
                    // 刷新数据
                    this.loadData();
                    // 提示结果
                    this.$message({
                        type: 'success',
                        message: '删除成功!'+id
                    });
                })

            
           })
        },
        closeModalHandler(){
           this.visible=false;
        },
        toUpdateHandler(row){
            // 在模态框中显示出当前行信息
            this.form=row;

           this.title="修改员工信息";
           this.visible=true;
        },
        toAddHandler(){
            // 将form变为初始值
            this.form={
                type:"order"
            }

           this.title="录入员工信息";
           this.visible=true;
        }
    },
    // 用于存放要想向网页中显示的数据
    data(){
        return{
            visible:false,
            order:[],
            form:{
                type:"order"
            },
            
            params:{
              page:0,
              pageSize:10
            },
            title:"ss"
        }        
    },
    created(){
        // this为当前
    //  vue实例创建完毕 
            this.loadData();
       
    }
}
</script>

<style scoped>

</style>