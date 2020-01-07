<template>
    <div>
        <el-button type="success" size="small" @click="toAddHendler">添加</el-button>
        <el-button type="danger" size="small">批量删除</el-button>
        <!-- 表格 -->
        <el-table :data="product">
            <el-table-column prop="id" label="编号"></el-table-column>
            <el-table-column prop="name" label="产品名称"></el-table-column>
            <el-table-column prop="price" label="价格"></el-table-column>
            <el-table-column prop="description" label="描述"></el-table-column>
            <el-table-column prop="categoryId" label="所属栏目"></el-table-column>
            <el-table-column prop="status" label="状态"></el-table-column>
            <el-table-column width="200" prop="photo" label="产品照片">
                <template   slot-scope="scope">            
                      <img :src="scope.row.photo"  min-width="70" height="70" />
                </template>   
            </el-table-column>
         
            <el-table-column fixed="right" label="操作">
                <template v-slot="slot">
                    <a href="" @click.prevent="toDeleteHandler(slot.row.id)">删除</a>
                    <a href="" @click.prevent="toUpdateHandler(slot.row)">修改</a>
                </template>
            </el-table-column>
        </el-table>
        <!-- 分页 -->
        <el-pagination layout="prev, pager, next" :total="50"></el-pagination>
        <!-- 模态框 -->
        <el-dialog  :title="title" :visible.sync="visible" width="60%">
            {{form}}
            <el-form :model="form" labei-width="80px"> 
               
               <el-form-item label="产品名称">       
                 <el-input type="name" v-model="form.name"></el-input>
               </el-form-item>
               <el-form-item label="价格">       
                 <el-input v-model="form.price"></el-input>
               </el-form-item>
               <el-form-item label="描述">       
                 <el-input v-model="form.description"></el-input>
               </el-form-item>
               <el-form-item label="所属栏目">       
                 <el-select v-model="form.categoryId">
                     <el-option v-for="item in options" :key="item.id" :label="item.name" :value="item.id">                        
                     </el-option>
                 </el-select>
               </el-form-item>  
               <el-form-item label="照片">
                   <el-upload
                    class="upload-demo"
                        action="http://134.175.154.93:6677/file/upload"
                        :on-success="uploadSuccessHandler"
                        :file-list="fileList"
                        list-type="picture">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-form-item>    
            </el-form>

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
    methods:{
        uploadSuccessHandler(){
            
            let photo="http://134.175.154.93:8888/group1/"+response.data.id
            //将图片地址设置到form便于一起提交给后台
            this.form.photo=photo;
        
         
        },
        loadCategory(){
            let url = "http://localhost:6677/category/findAll"
            request.get(url).then((response)=>{
                // 将查询结果设置到products中，this指向外部函数的this
                this.options = response.data;
      })
    },

        loadData(){
            let url="http://localhost:6677/product/findAll"
            request.get(url).then((response)=>{
               this.product=response.data;
            })
            
        },

        submitHandler(){
            // this.form 对象--字符串-->后台
            // 通过request与后台交互，并且携带参数
            let url="http://localhost:6677/product/saveOrUpdate";
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
        toAddHendler(){
             this.form={
                type:"product"
            }
            this.title="录入产品信息";
            this.visible=true;
        },
         toDeleteHandler(id){
           this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
                //   调用后台接口完成删除
                let url="http://localhost:6677/product/deleteById?id="+id;
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
        toUpdateHandler(row){
            this.form=row;
            this.title="修改产品信息";
            this.visible=true;
        },
        closeModalHandler(){
            this.visible=false;
        },



    },
    data(){
        return{
           fileList:[],
           options:[],
           visible:false,
           product:[],
           form:{
               type:"porduct"
           },
           title:'测试'
           
        }    
    },
    created(){
        this.loadData();
        // 加载栏目信息,用于表中下拉菜单
        this.loadCategory();
    }
}


</script>


<style scoped>
