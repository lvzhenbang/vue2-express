<template>
  <div>
    <el-menu theme="dark" class="el-menu-demo" mode="horizontal">
      <el-menu-item index="1">电影库</el-menu-item>
      <el-button class="btn" type="primary" icon="plus" @click="handleAdd"></el-button>
    </el-menu>
    <el-table
      v-loading.body="loading"
      :data="movies"
      @row-click="rowLink"
      border
      style="width: 100%">
      <el-table-column
        label="电影海报"
        align="center"
        >
        <template scope="movie">
          <el-row type="flex" class="row-bg" justify="center" align="middle">
            <img :src="movie.row.poster" alt="">
          </el-row>
        </template>
      </el-table-column>
      <el-table-column
        label="电影名称"
        align="center"
        >
        <template scope="movie">
          <p>{{ movie.row.title }}</p>
        </template>
      </el-table-column>
      <el-table-column
        label="评分"
        align="center"
        >
        <template scope="movie">
          <p>{{ movie.row.rating}}</p>
        </template>
      </el-table-column>
      <el-table-column
        label="类型"
        align="center"
        >
        <template scope="movie">
          <el-tag 
            class="tag" 
            v-for="(item,index) in movie.row.genres" 
            key="index">
              {{item}}
            </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template scope="movie">
          <el-button
            size="small"
            type="primary"
            icon="edit"
            @click.stop="handleEdit(movie.$index, movie.row)"></el-button>
          <el-button
            size="small"
            type="danger"
            icon="delete"
            @click.stop="handleDelete(movie.$index,movie.row)"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 弹出层 -->
    <el-dialog title="电影详情" v-model="dialogFormVisible" size="small">
      <el-form label-width="120px" :model="movieInfo">
        <el-form-item label="电影名称">
          <el-input v-model="movieInfo.title"></el-input>
        </el-form-item>
        <el-form-item label="海报地址-小图">
          <el-input v-model="movieInfo.poster"></el-input>
        </el-form-item>
          <el-form-item v-show="isAdd" label="海报地址-大图">
            <el-input v-model="movieInfo.poster_lg"></el-input>
          </el-form-item>
        <el-form-item label="类型">
          <el-input v-model="genres" placeholder="请用空格分开输入类型"></el-input>
        </el-form-item>
        <el-form-item label="评分">
          <el-input v-model="movieInfo.rating"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button v-show="!isAdd" type="primary" @click="saveMovie(movieInfo)">确 定</el-button>
        <el-button v-show="isAdd" type="primary" @click="addMovie(movieInfo)">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        loading:true,
        dialogFormVisible: false,
        movies: [] ,
        movieInfo: {
          title: '',
          poster:'',
          poster_lg:'',
          genres: [],
          rating: ''
        },
        genres:'',
        isAdd: false
      }
    },
    created(){
        this.getMovies();
        this.loading = false
    },
    computed:{
      
    },
    methods: {
        // 获取所有电影
        getMovies(){
          this.$http.get('/api/movie')
              .then(res => {
                  // console.log('获取数据成功：')
                  // console.dir(res.data)
                  this.movies = res.data
              })
              .catch(err => {
                  this.$message.error(`${err.message}`, 'ERROR!')
                  console.log(err)
              })
        },
        // 增加单条电影
        addMovie(){
          let movie = this.movieInfo;
          movie.genres = this.genres.split(' ');
          let formData = {
            title: movie.title,
            poster: movie.poster,
            rating: movie.rating,
            genres: movie.genres
          };
          this.movies.unshift(formData);
          this.$http.post('/api/movie',formData)
              .then(res => {
                  console.dir(res.data)
              })
              .catch(err => {
                  this.$message.error(`${err.message}`, 'ERROR!')
                  console.log(err)
              })
          this.dialogFormVisible = false;
          this.$notify({
            title: '添加成功',
            message: `${movie.title}已成功添加并录入电影库`,
            type: 'success'
          });
        },
        // 编辑保存更新电影
        saveMovie(movie){
          this.dialogFormVisible = false;
          let formData = {
            title: movie.title,
            poster: movie.poster,
            rating: movie.rating,
            genres: movie.genres
          };
          this.$http.put(`/api/movie/${movie._id}`,formData)
            .then(res => {
                console.dir(res.data)
            })
            .catch(err => {
                this.$message.error(`${err.message}`, 'ERROR!')
                console.log(err)
            })
          // 保存新数据的同时，触发视图更新
          this.movies.splice(movie.index,1,formData);
          this.$notify({
            title: '修改成功',
            message: `${movie.title}已成功修改并改录入电影库`,
            type: 'success'
          });
        },
        handleEdit(index, row) {
          this.genres = row.genres.join(' ');
          this.dialogFormVisible = true;
          this.movieInfo = Object.assign({index:index},row);
        },
        // 添加按钮的事件
        handleAdd(){
          this.movieInfo = {};
          this.genres = "";
          this.isAdd = true;
          this.dialogFormVisible = true;
        },
        handleDelete(index,movie) {
          this.dialogFormVisible = false;
          this.$notify({
            title: '删除成功',
            message: `${movie.title}已从电影库中成功删除`,
            type: 'success'
          });
          this.movies = this.movies.filter((item,key)=> key !== index)
          this.$http.delete(`/api/movie/${movie._id}`)
              .then(res => {
                  console.dir(res.data)
              })
              .catch(err => {
                  this.$message.error(`${err.message}`, 'ERROR!')
                  console.log(err)
              })
        },
        rowLink(row, event, column){
          this.$router.push({ name: 'detail', params: { movieId: row._id }})
        }
    }
  }
</script>
<style>
  .tag{
    margin-right: 5px;
  }
  .el-button.el-button {
    margin: 5px 10px;
  }
  .el-button.el-button.btn{
    float:right;
    margin-top:12px;
    margin-bottom:12px;
  }
  @media screen and (max-width:640px){
    .el-dialog.el-dialog--small{
      width: 90%
    }
    .el-table thead .cell, .el-table thead th>div{
      padding-left:0;
      padding-right:0
    }
    .el-table tbody .cell{
      padding: 6px 0;
    }
  }
</style>