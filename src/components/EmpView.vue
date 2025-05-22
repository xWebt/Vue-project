<template>
    <div class="common-layout"> <el-container class="full-height-container">
            <el-header class="main-header">
                <div class="header-content"> tlias 智能学习辅助系统
                </div>
            </el-header>

            <el-container class="content-container">
                <el-aside width="200px" class="main-aside">
                    <el-scrollbar>
                        <el-menu :default-openeds="['1']" router>
                            <el-sub-menu index="1">
                                <template #title>
                                    <el-icon>
                                        <Message />
                                    </el-icon>系统信息管理
                                </template>
                                <el-menu-item index="/department">部门管理</el-menu-item>
                                <el-menu-item index="/employee">员工管理</el-menu-item>
                            </el-sub-menu>
                        </el-menu>
                    </el-scrollbar>
                </el-aside>

                <el-main class="main-content-area"> <el-scrollbar class="main-scrollbar"> <el-form :inline="true"
                            :model="formInline" class="search-form"> <el-form-item label="姓名">
                                <el-input v-model="formInline.name" placeholder="请输入姓名" clearable />
                            </el-form-item>
                            <el-form-item label="性别">
                                <el-select v-model="formInline.gender" placeholder="请选择性别" clearable> <el-option
                                        label="男" value="1" />
                                    <el-option label="女" value="2" />
                                </el-select>
                            </el-form-item>
                            <el-form-item label="入职时间">
                                <el-date-picker v-model="formInline.entryDateRange" type="daterange" range-separator="至"
                                    start-placeholder="开始日期" end-placeholder="结束日期" :size="pickerSize"
                                    value-format="YYYY-MM-DD" clearable />
                            </el-form-item>
                            <el-form-item>
                                <el-button type="primary" @click="onSubmit">查询</el-button>
                                <el-button @click="onReset">重置</el-button> </el-form-item>
                        </el-form>

                        <el-table :data="tableData" style="width: 100% " border stripe class="data-table">
                            <el-table-column prop="name" label="姓名" width="180" />
                            <el-table-column prop="image" label="图像" width="180">
                                <template #default="scope">
                                    <img :src="scope.row.image" alt="图像" class="employee-image">
                                </template>
                            </el-table-column>
                            <el-table-column prop="gender" label="性别" width="140"> <template #default="scope">
                                    {{ scope.row.gender === 1 ? '男' : '女' }} </template>
                            </el-table-column>
                            <el-table-column prop="job" label="职位" width="140" /> <el-table-column prop="entrydate"
                                label="入职日期" width="180" />
                            <el-table-column prop="updatetime" label="最后操作时间" width="230" /> <el-table-column label="操作"
                                fixed="right" width="160"> <template #default="scope"> <el-button type="primary"
                                        size="small" @click="handleEdit(scope.row)">编辑</el-button> <el-button
                                        type="danger" size="small" @click="handleDelete(scope.row)">删除</el-button>
                                </template>
                            </el-table-column>
                        </el-table>
                        <el-pagination background layout="total, sizes, prev, pager, next, jumper" :total="total"
                            :page-sizes="[10, 20, 30, 40]" :page-size="pageSize" v-model:current-page="currentPage"
                            @size-change="handleSizeChange" @current-change="handleCurrentChange"
                            class="pagination-container">
                        </el-pagination>
                    </el-scrollbar>
                </el-main>
            </el-container>
        </el-container>
    </div>
</template>

<script setup lang="ts">
import { Menu as Message } from '@element-plus/icons-vue'
import { reactive, ref, onMounted } from 'vue' 
// import axios from 'axios'; 

const currentPage = ref(1);
const pageSize = ref(10); 
const total = ref(0); 

interface Employee {
    name: string;
    image: string;
    gender: 1 | 2; // 1:男, 2:女
    job: string;
    entrydate: string;
    updatetime: string;
}

function generateEmployeeData(count: number): Employee[] {
    const employees: Employee[] = [];
    const names = ['张三', '李四', '王五', '赵六', '孙七', '周八', '吴九', '郑十', '陈一', '朱二'];
    const jobs = ['软件工程师', '产品经理', '数据分析师', 'UI设计师', '前端开发', '后端开发', '测试工程师', '项目经理', '运维工程师', '市场专员'];
    const genders: (1 | 2)[] = [1, 2];

    const baseImageUrl = 'https://picsum.photos/50/50?random=';

    for (let i = 0; i < count; i++) {
        const randomName = names[Math.floor(Math.random() * names.length)];
        const randomGender = genders[Math.floor(Math.random() * genders.length)];
        const randomJob = jobs[Math.floor(Math.random() * jobs.length)];

        const entryDate = new Date(Date.now() - Math.random() * 2 * 365 * 24 * 60 * 60 * 1000);
        const updateTime = new Date(Date.now() - Math.random() * 30 * 24 * 60 * 60 * 1000);

        employees.push({
            name: randomName + String.fromCharCode(65 + (i % 26)),
            image: `${baseImageUrl}${i + 1}`,  
            gender: randomGender,
            job: randomJob,
            entrydate: entryDate.toISOString().split('T')[0],
            updatetime: updateTime.toISOString().replace('T', ' ').substring(0, 19),
        });
    }
    return employees;
}

const tableData = ref<Employee[]>([]); 

const formInline = reactive({
    name: '',
    gender: '', 
    entryDateRange: [] as [Date, Date] | null, 
});

const pickerSize = ref('default'); 

const getEmpList = async () => {
    try {
        const params: any = {
            name: formInline.name,
            gender: formInline.gender,
            page: currentPage.value,
            pageSize: pageSize.value
        };

        if (formInline.entryDateRange && formInline.entryDateRange.length === 2) {
            params.begin = formInline.entryDateRange[0].toISOString().split('T')[0];
            params.end = formInline.entryDateRange[1].toISOString().split('T')[0];
        }

        const mockData = generateEmployeeData(50); 
        total.value = mockData.length;

        const start = (currentPage.value - 1) * pageSize.value;
        const end = start + pageSize.value;
        tableData.value = mockData.slice(start, end);

    } catch (error) {
        console.error("获取员工数据失败:", error);
        tableData.value = [];
        total.value = 0;
    }
}


const onSubmit = () => {
    currentPage.value = 1;
    getEmpList();
}

const onReset = () => {
    formInline.name = '';
    formInline.gender = '';
    formInline.entryDateRange = null;
    currentPage.value = 1;
    getEmpList(); 
}

const handleSizeChange = (val: number) => {
    pageSize.value = val;
    currentPage.value = 1; 
    getEmpList();
}
const handleCurrentChange = (val: number) => {
    currentPage.value = val;
    getEmpList();
}

const handleEdit = (row: Employee) => {
    console.log('编辑行:', row);
}

const handleDelete = (row: Employee) => {
    console.log('删除行:', row);
}


onMounted(() => {
    getEmpList();
})

</script>

<style scoped>
.common-layout {
    height: 100vh;
    display: flex;
    flex-direction: column;
}

.full-height-container {
    flex: 1;
    overflow: hidden;
}

/* 主页眉 */
.main-header {
    background-color: rgb(238, 241, 246);
    color: var(--el-text-color-primary);
    display: flex;
    align-items: center;
    padding: 0 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    z-index: 100;
}

.header-content {
    font-size: 24px;
    font-weight: bold;
    color: #333;
}

.content-container {
    flex: 1;
}

.main-aside {
    background-color: #fff;
    border-right: 1px solid #e6e6e6;
    box-shadow: 2px 0 4px rgba(0, 0, 0, 0.05);
}

.main-aside .el-menu {
    border-right: none;
    min-height: 100%;
}

.main-content-area {
    background-color: #f0f2f5;
    padding: 20px;
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.main-scrollbar {
    height: 100%;
}


.search-form {
    background-color: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    display: flex;
    flex-wrap: wrap;
    gap: 15px;
    justify-content: flex-start;
}

.search-form .el-input,
.search-form .el-select,
.search-form .el-date-picker {
    width: 200px;
}

.data-table {
    background-color: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.employee-image {
    width: 50px;
    height: 50px;
    object-fit: cover;
    border-radius: 4px;
}

.pagination-container {
    margin-top: 20px;
    display: flex;
    justify-content: flex-end;
}
</style>