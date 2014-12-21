---
layout:     post
title:      convnet源码阅读 - Matrix类
category:   post
description: convnet Matrix类的部分源码剖析
tags: 开源项目 深度学习 CNN 源码阅读
---

# convnet源码阅读 - Matrix类

## 简介
**Matrix**是一个GPU矩阵类，封装了各种GPU计算和GPU之间数据的传输。通过Matrix类，convnet完成了C++代码和cuda代码的隔离，在涉及到逻辑和算法的代码部分只需要调用Matrix类的接口函数，而不需要在意具体实现。

Matrix类的实现来源于cudamat，这是一个python的矩阵库。cudamat为了与cuBLAS兼容，矩阵为**column-major**，而不像一般的C代码一样为row-major。所谓column-major是指每列元素在内存中相邻，内存中排列的先后顺序为第一列、第二列直到第N列。

由于Matrix为完成实际计算的结构，对理解算法完成的功能和多GPU的实现很重要。

## 数据成员
私有的数据成员变量有：

	cudamat mat_, mat_t_; // 具体的矩阵结构及其转置
	cudaEvent_t ready_; // ?
	Shape4D shape_;
	int gpu_id_; // 完成这个matrix实际计算的gpu id
	string name_; // 为这个matrix指定的名字

静态成员变量有：

	static int num_boards_; // 
	static int current_gpu_id_; // 静态变量，所有的matrix共享相同的gpu id
	static vector<int> boards_;
	static vector<size_t> temp_size_, ones_size_;
	static vector<Matrix> ones_, temp_;
	static vector<rnd_struct> rnd_;
	static map<string, long> gpu_memory_;
	static Matrix rgb_to_yuv_mat_;
这些静态变量为所有Matrix实例共有，大部分为了支持多GPU通讯，另有部分如ones_、temp_为了避免重复分配内存。

cudamat结构为在cudamat库中，为完成实际运算的主要部分，包括：

	struct cudamat {
	    float* data_host; // matrix在host上的数据
	    float* data_device; // matrix在device上的数据
	    int on_device; // ??
	    int on_host;
	    int size[2]; // matrix的大小，size[0]为行，size[1]为列
	    int is_trans; // 0 or 1，标识是否为转置
	    int owns_data; // 标识目前是否已包含数据
	    cudaTextureObject_t tex_obj;
	};


## 数据传输及GPU间通信
数据的传输涉及host和device通信以及device之间的通信（即GPU之间）。
### 将mat_的内存从host拷贝到device
调用关系为：

	Matrix::CopyToDevice
	->
	Matrix::CopyToDeviceSlice // 将指定范围[start, end)内的列从host拷贝到device
	-> 
	copy_to_device_slice 
	->
	cublasSetVector // CUBLAS库的函数，从mat_->data_host到mat_->data_device

### 将mat_的内存从device拷贝到host
	Matrix::CopyToHost 
	-> 
	Matrix::CopyToHostSlice
	->
	copy_to_host_slice
	-> 
	cublasGetVector // 从mat_->data_device到mat_->data_host

### 主存间数据的传输
	Matrix::CopyFromMainMemory
	-> 
	memcpy
### GPU间的数据传输
	Matrix::Set(Matrix& val)
	->
	copy_on_device
	->
	cudaMemcpy
可见在convnet中不同GPU的数据传输通过Set函数完成，但在调用Set之前必须通过进行必要的设置。Matrix::SetupCUDADevices会调用cuda_set_P2P，这个函数中完成了cuda中GPU点对点通信的设置。

	int cuda_set_P2P(int gpu1, int gpu2) {
	  bool is_fermi = cuda_is_fermi(gpu1) && cuda_is_fermi(gpu2);
	  
	  int access2from1, access1from2;
	
	  // 确定GPU之间是否可以直接访问相互的内存
	  cudaDeviceCanAccessPeer(&access2from1, gpu1, gpu2); 
	  cudaDeviceCanAccessPeer(&access1from2, gpu2, gpu1);
	
	  // 令gpu1和gpu2互相之间可以直接访问
	  if(is_fermi && same_complex) {
		// 设置当前GPU为gpu1
	    cudaSetDevice(gpu1);
        // 允许当前device可以直接访问peer device，即gpu2
	    cudaDeviceEnablePeerAccess(gpu2, 0); //second argument is flags
	    cudaSetDevice(gpu2);
	    cudaDeviceEnablePeerAccess(gpu1, 0); //second argument is flags
	    return 0;
	  } else {
	    return CUDA_ERROR;
	  }
	}
通过上述设置，如果gpu支持uva，那么当前gpu可以直接访问peer device分配的内存，否则需要调用cudaPeerRegister等函数。convnet假定GPU都支持uva，因此接下来通过Matrix::Set直接使用cudaMemcpy就可完成GPU间数据的传送。


## 矩阵操作及数学计算 
### Matrix::AddRowVec
	Matrix::AddRowVec
	->
	add_row_vec(mat, vec, target)
假设mat的shape为(10,20),vec为(1,20)，则计算结果target为(10,20)，即mat的第i列每行元素与vec[i]相加。Matrix::AddColumnVec有着相似的作用，只是变为对列操作。

### Matrix::SumRows及Matrix::SumCols
这两个函数均调用sum_by_axis(mat， target， axis)

	A - shape is (10,20)
	col_sums - shape is (1,20)
	row_sums - shape is (10,1)
	
	Matrix::SumRows -> sum_by_axis(A, col_sums, 0)	// 对每列求和
	Matrix::SumCols -> sum_by_axis(A, row_sums, 1)	// 对每行求和
	
Matrix还有其他很多相关接口函数，如Sqrt()、Norm()、Divide()等，实现了一般矩阵类常见的操作。

## 其他

### Matrix::SetDeviceSetDevice
	Matrix::SetDeviceSetDevice
	-> 
	cuda_set_device
	->
	cudaSetDevice // 表示当前要使用哪个gpu，调用后内存分配、计算都在这个gpu中完成

### Matrix::SyncAllDevices
	// 同步所有的device
	void Matrix::SyncAllDevices() {
	  int current_gpu_backup = current_gpu_id_;
	  for (int i = 0; i < num_boards_; i++) {
		// 选择device i为当前的GPU
	    SetDevice(i);
		// 阻塞，直到当前GPU中的task完成
	    cuda_sync_threads();
	  }
	  // 恢复原gpu
	  SetDevice(current_gpu_backup);
	}