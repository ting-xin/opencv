# Parallel
parallel_for_ 的基类[ParallelLoopBody](../include/opencv2/core/utility.hpp)

## 使用方法
> 声明一个类并继承自基类ParallelLoopBody，然后重写operator()方法。

```c++
class parallelConvolution : public ParallelLoopBody
{
private:
    Mat m_src, &m_dst;
    Mat m_kernel;
    int sz;
 
public:
    parallelConvolution(Mat src, Mat &dst, Mat kernel)
        : m_src(src), m_dst(dst), m_kernel(kernel)
    {
        sz = kernel.rows / 2;
    }
 
    virtual void operator()(const Range &range) const CV_OVERRIDE
    {
        for (int r = range.start; r < range.end; r++)
        {
            int i = r / m_src.cols, j = r % m_src.cols;
 
            double value = 0;
            for (int k = -sz; k <= sz; k++)
            {
                uchar *sptr = m_src.ptr(i + sz + k);
                for (int l = -sz; l <= sz; l++)
                {
                    value += m_kernel.ptr<double>(k + sz)[l + sz] * sptr[j + sz + l];
                }
            }
            m_dst.ptr(i)[j] = saturate_cast<uchar>(value);
        }
    }
};
```

> 在C++11之后，可以使用lambda表达式来更简单的实现并行操作
```c++
    parallel_for_(Range(0, rows * cols), [&](const Range &range)
                    {
                        for (int r = range.start; r < range.end; r++)
                        {
                            int i = r / cols, j = r % cols;
 
                            double value = 0;
                            for (int k = -sz; k <= sz; k++)
                            {
                                uchar *sptr = src.ptr(i + sz + k);
                                for (int l = -sz; l <= sz; l++)
                                {
                                    value += kernel.ptr<double>(k + sz)[l + sz] * sptr[j + sz + l];
                                }
                            }
                            dst.ptr(i)[j] = saturate_cast<uchar>(value);
                        }
                    });
```