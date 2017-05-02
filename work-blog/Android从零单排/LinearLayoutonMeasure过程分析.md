## LinearLayout onMeasure 过程分析
>昨天做一个LinearLayout布局的时候，遇到一个想象，当给LinearLayout设置android:background属性的时候，LinearLayout的高度就出现意想不到的效果。

### 一、导火索
先看我昨天的测试布局文件：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:orientation="vertical"
	    android:id="@+id/parent"
	    android:background="@mipmap/common_bg_day">
	    <LinearLayout
	        android:layout_height="wrap_content"
	        android:layout_width="wrap_content"
	        android:id="@+id/child"
	        android:background="@mipmap/setting_list_bg_day">
	        <Button
	            android:id="@+id/button"
	            android:layout_width="wrap_content"
	            android:layout_height="88px"
	            android:text="侧四软件的大小"
	            ></Button>
	    </LinearLayout>
	
	</LinearLayout>

针对两个LinearLayout的高度，我同意设置为wrap_content，但是给LinearLayout都设置了背景。最后打印出来的高度：

	D/Test: parent:239 child:170   button:88

一时间没搞懂这个问题，所以决心Look下LinearLayout的源码看看。

>LinearLayout是我们Android中最常见的布局，里面有的属性确实知悉的比较少，这次也算探究一下。
>源码Android-25

### 二、LinearLayout测量过程分析

#### onMeasure方法

----------
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	    if (mOrientation == VERTICAL) {
	        measureVertical(widthMeasureSpec, heightMeasureSpec);
	    } else {
	        measureHorizontal(widthMeasureSpec, heightMeasureSpec);
	    }
    }

在**onMeasure**方法中，根据**LinearLayout**的方向调用不同的**measure**方法进行测量。


#### measureVertical()

----------
	void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
		//用于记录测量过程中子控件相加的高度值
        mTotalLength = 0;
		//记录所有控件中的最大宽度值
        int maxWidth = 0;
        int childState = 0;
        int alternativeMaxWidth = 0;
		//子控件中layout_weight>0的View的最大宽度
        int weightedMaxWidth = 0;
		//子控件是否为match_parent的标志位，用于判断是否需要重新测量
        boolean allFillParent = true;
		//所有子控件的weight之和
        float totalWeight = 0;
		// 如您所见，得到所有子控件的数量，准确的说，它得到的是所有同级子控件的数量
        // 在官方的注释中也有着对应的例子
        // 比如TableRow，假如TableRow里面有N个控件，而LinearLayout（TableLayout也是继承LinearLayout哦）下有M个TableRow，那么这里返回的是M，而非M*N
        // 但实际上，官方似乎也只是直接返回getChildCount()，起这个方法名的原因估计是为了让人更加的明白，毕竟如果是getChildCount()可能会让人误认为为什么没有返回所有（包括不同级）的子控件数量
        final int count = getVirtualChildCount();
        //宽度和高度的测量模式
        final int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        final int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        boolean matchWidth = false;
        boolean skippedMeasure = false;

        final int baselineChildIndex = mBaselineAlignedChildIndex;        
        final boolean useLargestChild = mUseLargestChild;

        int largestChildHeight = Integer.MIN_VALUE;
        int consumedExcessSpace = 0;

        // See how tall everyone is. Also remember max width.
		//第一次测量，遍历所有的子View的高度，同时记录下最大的高度
        for (int i = 0; i < count; ++i) {
			......
        }

		//如果LinearLayout所有子view的总高度大于0，并且使用了Divider，则加上Divider的高度
        if (mTotalLength > 0 && hasDividerBeforeChildAt(count)) {
            mTotalLength += mDividerHeight;
        }

        if (useLargestChild &&
                (heightMode == MeasureSpec.AT_MOST || heightMode == MeasureSpec.UNSPECIFIED)) {
			//如果设置了measureWithLargestChild 且 高度测量模式为 AT_MOST 或者 UNSPECIFIED，则重新计算 mToatalLength
            mTotalLength = 0;
            for (int i = 0; i < count; ++i) {
                .......
            }
        }

        // Add in our padding
		//添加child view的padding值
        mTotalLength += mPaddingTop + mPaddingBottom;

        int heightSize = mTotalLength;

        // Check against our minimum height
		//检查我们的高度值跟最小的高度值进行比较，取二者中较大者
        heightSize = Math.max(heightSize, getSuggestedMinimumHeight());
        
        // Reconcile our calculated size with the heightMeasureSpec
        int heightSizeAndState = resolveSizeAndState(heightSize, heightMeasureSpec, 0);
        heightSize = heightSizeAndState & MEASURED_SIZE_MASK;
        
        // Either expand children with weight to take up available space or
        // shrink them if they extend beyond our current bounds. If we skipped
        // measurement on any children, we need to measure them now.
		//此处开始遍历测量那些设置了weight属性的控件，同时开始计算我们跳过计算的控件
        int remainingExcess = heightSize - mTotalLength
                + (mAllowInconsistentMeasurement ? 0 : consumedExcessSpace);
        if (skippedMeasure || remainingExcess != 0 && totalWeight > 0.0f) {
            float remainingWeightSum = mWeightSum > 0.0f ? mWeightSum : totalWeight;

			......
        } else {
            ......
        }

        if (!allFillParent && widthMode != MeasureSpec.EXACTLY) {
            maxWidth = alternativeMaxWidth;
        }
        
        maxWidth += mPaddingLeft + mPaddingRight;

        // Check against our minimum width
        maxWidth = Math.max(maxWidth, getSuggestedMinimumWidth());
        
        setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
                heightSizeAndState);

        if (matchWidth) {
            forceUniformWidth(count, heightMeasureSpec);
        }
    }

在measureVertical方法中，总体来说分为两部分，一部分是针对子view的基础属性测量，另一部分是对weight部分进行计算。

#### 第一次测量过程
	// See how tall everyone is. Also remember max width.
	//第一次测量，遍历所有的子View的高度，同时记录下最大的高度
	for (int i = 0; i < count; ++i) {
		final View child = getVirtualChildAt(i);
		
		//如果child==null，则通过measureNullChild方法获取对应的高度值。
		//我们查看该方法源码发现该方法返回常量0，通过continue直接跳过
		if (child == null) {
			mTotalLength += measureNullChild(i);
			continue;
		}
		
		//如果child的visible属性为View.GONE，则通过getChildrenSkipCount方法计算
		//跳过测量的view个数，getChildrenSkipCount返回常量0，通过continue直接跳过
		if (child.getVisibility() == View.GONE) {
		   i += getChildrenSkipCount(child, i);
		   continue;
		}
	
		//判断在当前view上是否有divider，如果有则将Divider的高度添加到总高度上
		if (hasDividerBeforeChildAt(i)) {
			mTotalLength += mDividerHeight;
		}
	
		//获取当前子view的LayoutParams属性，统计weight属性
		final LayoutParams lp = (LayoutParams) child.getLayoutParams();
		totalWeight += lp.weight;
		//判断当前view的height和weight属性，判断当前view是否是使用weight进行计算
		//这里有一个疑惑：weight的属性为什么非要判断lp.height==0，因为在我们实际的使用过程中不一定非要为0
		final boolean useExcessSpace = lp.height == 0 && lp.weight > 0;
		if (heightMode == MeasureSpec.EXACTLY && useExcessSpace) {
			// 这个if里面需要满足三个条件：
			// * LinearLayout的高度为match_parent(或者有具体值)
			// * 子控件的高度为0
			// * 子控件的weight>0
			// 这其实就是我们通常情况下用weight时的写法
			// 测量到这里的时候，会给个标志位，稍后再处理。此时会计算总高度
			final int totalLength = mTotalLength;
			//累加上当前view的margin属性
			mTotalLength = Math.max(totalLength, totalLength + lp.topMargin + lp.bottomMargin);
			skippedMeasure = true;
		} else {
			/*这里的情形分的就比较多：
			 *1、LinearLayout的高度为wrap_content或不定值，但子控件lp.height == 0 && lp.weight > 0
			 *2、子控件的height>0，此时weight>=0
			 *3、LinearLayout的高度为wrap_content但是子控件的height>0		
			
			//子控件高度为0，并且weight>0
			if (useExcessSpace) {
				//出现上述情况1，将子view的height指定为WRAP_CONTENT
				lp.height = LayoutParams.WRAP_CONTENT;
			}
	
			// Determine how big this child would like to be. If this or
			// previous children have given a weight, then we allow it to
			// use all available space (and we will shrink things later
			// if needed).
			//判断统计到当前的总weight是否=0，
			//totalWeight==0则说明统计到当前的控件大小都是已知，则已用的高度=mTotalLength。
			//反之，则useHeight=0
			final int usedHeight = totalWeight == 0 ? mTotalLength : 0;
			//计算子控件宽高德MeasureSpec参数
			measureChildBeforeLayout(child, i, widthMeasureSpec, 0,
					heightMeasureSpec, usedHeight);
	
			final int childHeight = child.getMeasuredHeight();
			if (useExcessSpace) {
				//如果子控件height=0且weight>0
				lp.height = 0;
				consumedExcessSpace += childHeight;
			}
	
			final int totalLength = mTotalLength;
			mTotalLength = Math.max(totalLength, totalLength + childHeight + lp.topMargin +
				   lp.bottomMargin + getNextLocationOffset(child));
			//如果使用最大高度，则记录下所有子View中的最大高度
			if (useLargestChild) {
				largestChildHeight = Math.max(childHeight, largestChildHeight);
			}
		}
	
		/**
		 * If applicable, compute the additional offset to the child's baseline
		 * we'll need later when asked {@link #getBaseline}.
		 */
		 //如果设置最后一个View作为baseLine基线
		if ((baselineChildIndex >= 0) && (baselineChildIndex == i + 1)) {
		   mBaselineChildTop = mTotalLength;
		}
	
		// if we are trying to use a child index for our baseline, the above
		// book keeping only works if there are no children above it with
		// weight.  fail fast to aid the developer.
		//在设置的baselineChildIndex之前有个View设置了weight属性
		//这是就会提示下面的错误，无法成功。这是为什么呢？
		if (i < baselineChildIndex && lp.weight > 0) {
			throw new RuntimeException("A child of LinearLayout with index "
					+ "less than mBaselineAlignedChildIndex has weight > 0, which "
					+ "won't work.  Either remove the weight, or don't set "
					+ "mBaselineAlignedChildIndex.");
		}
	
		boolean matchWidthLocally = false;
		//当LinearLayout的宽度不是EXACTLY或者确定值
		//并且当前子空间的额宽度是MATCH_PARENT的时候
		if (widthMode != MeasureSpec.EXACTLY && lp.width == LayoutParams.MATCH_PARENT) {
			// The width of the linear layout will scale, and at least one
			// child said it wanted to match our width. Set a flag
			// indicating that we need to remeasure at least that view when
			// we know our width.
			matchWidth = true;
			matchWidthLocally = true;
		}
	
		final int margin = lp.leftMargin + lp.rightMargin;
		final int measuredWidth = child.getMeasuredWidth() + margin;
		maxWidth = Math.max(maxWidth, measuredWidth);
		childState = combineMeasuredStates(childState, child.getMeasuredState());
	
		allFillParent = allFillParent && lp.width == LayoutParams.MATCH_PARENT;
		if (lp.weight > 0) {
			/*
			 * Widths of weighted Views are bogus if we end up
			 * remeasuring, so keep them separate.
			 */
			weightedMaxWidth = Math.max(weightedMaxWidth,
					matchWidthLocally ? margin : measuredWidth);
		} else {
			alternativeMaxWidth = Math.max(alternativeMaxWidth,
					matchWidthLocally ? margin : measuredWidth);
		}
	
		//getChildrenSkipCount一直返回常量0
		i += getChildrenSkipCount(child, i);
	}

在这次的测量中，这部分代码的核心就是围绕着mTotalLenght的计算而展开的，代表着计算出的LinearLayout的总长度。这部分高度值包含：子View的高度、子View对应的上下margin之和、divider的高度。

在上面的测量过程中，我们可以发现，针对View.GONE属性的View，无需对它的高度进行测量，可以直接跳过，这也是我们设置View.GONE可以隐藏控件占据空间的原因，也体现与View.INVISIBLE的区别。在这里，我们需要注意一下对lp.height==0&&lp.weight==0属性的View的处理，这部分处理分为两部分，一部分是LinearLayout的高度为EXACTLY的时候，通过将mTotalLenght与totalLength + lp.topMargin + lp.bottomMargin比较，取大值。

#### 第二次测量
    if (useLargestChild &&
            (heightMode == MeasureSpec.AT_MOST || heightMode == MeasureSpec.UNSPECIFIED)) {
		//如果设置了measureWithLargestChild 且 高度测量模式为 AT_MOST 或者 UNSPECIFIED，则重新计算 mToatalLength
        mTotalLength = 0;
		for (int i = 0; i < count; ++i) {
            final View child = getVirtualChildAt(i);
            if (child == null) {
                mTotalLength += measureNullChild(i);
                continue;
            }

            if (child.getVisibility() == GONE) {
                i += getChildrenSkipCount(child, i);
                continue;
            }

            final LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams)
                    child.getLayoutParams();
            // Account for negative margins
			//重新计算高度
            final int totalLength = mTotalLength;
            mTotalLength = Math.max(totalLength, totalLength + largestChildHeight +
                    lp.topMargin + lp.bottomMargin + getNextLocationOffset(child));
        }
    }

在第二次测量过程中，判断系统是否使用了largestChild属性，并且此时的LinearLayout高度是不定的，然后开始重新测量mTotalLenght的长度。

#### 第三次测量
		//计算未分配的宽度
	int remainingExcess = heightSize - mTotalLength
            + (mAllowInconsistentMeasurement ? 0 : consumedExcessSpace);
	//判断当前view是否需要计算weight
	//skippedMeasure = true 第一次计算进行表示了跳过计算的
	//remainingExcess != 0 && totalWeight > 0.0f，即剩余空间>0,并且总共的weight>0
    if (skippedMeasure || remainingExcess != 0 && totalWeight > 0.0f) {
		//计算剩余的weight属性
        float remainingWeightSum = mWeightSum > 0.0f ? mWeightSum : totalWeight;

        mTotalLength = 0;
		//遍历所有的view
        for (int i = 0; i < count; ++i) {
            final View child = getVirtualChildAt(i);
			//如果child == null或View.GONE时直接跳过。
            if (child == null || child.getVisibility() == View.GONE) {
                continue;
            }
			//获取view的LayoutParams属性
            final LayoutParams lp = (LayoutParams) child.getLayoutParams();
			//获取view的的weight属性
            final float childWeight = lp.weight;
			//如果view设置了weight属性
            if (childWeight > 0) {
				//通过比例计算出view所占的space大小
                final int share = (int) (childWeight * remainingExcess / remainingWeightSum);
				//计算出剩余space和weight大小
                remainingExcess -= share;
                remainingWeightSum -= childWeight;
				
                final int childHeight;
				//如果设置了useLarge属性并且LinearLayout的模式为MeasureSpec.EXACTLY
                if (mUseLargestChild && heightMode != MeasureSpec.EXACTLY) {
					//view的高度赋值为最大高度
                    childHeight = largestChildHeight;
                } else if (lp.height == 0 && (!mAllowInconsistentMeasurement
                        || heightMode == MeasureSpec.EXACTLY)) {
                    // This child needs to be laid out from scratch using
                    // only its share of excess space.
					//当子View的高度设置为0时，则分配计算出的share的大小。
                    childHeight = share;
                } else {
                    // This child had some intrinsic height to which we
                    // need to add its share of excess space.
					//如果子view有一定的高度，则原来的高度+空白高度。
                    childHeight = child.getMeasuredHeight() + share;
                }

				//计算子View的MeasureSpec变量
                final int childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(
                        Math.max(0, childHeight), MeasureSpec.EXACTLY);
                final int childWidthMeasureSpec = getChildMeasureSpec(widthMeasureSpec,
                        mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin,
                        lp.width);
                child.measure(childWidthMeasureSpec, childHeightMeasureSpec);

                // Child may now not fit in vertical dimension.
                childState = combineMeasuredStates(childState, child.getMeasuredState()
                        & (MEASURED_STATE_MASK>>MEASURED_HEIGHT_STATE_SHIFT));
            }

			//计算出子View对应的左右margin之和，用于计算宽度
            final int margin =  lp.leftMargin + lp.rightMargin;
			//计算宽度,将子控件的宽度+计算的margin值
            final int measuredWidth = child.getMeasuredWidth() + margin;
            maxWidth = Math.max(maxWidth, measuredWidth);

            boolean matchWidthLocally = widthMode != MeasureSpec.EXACTLY &&
                    lp.width == LayoutParams.MATCH_PARENT;

            alternativeMaxWidth = Math.max(alternativeMaxWidth,
                    matchWidthLocally ? margin : measuredWidth);

            allFillParent = allFillParent && lp.width == LayoutParams.MATCH_PARENT;

            final int totalLength = mTotalLength;
            mTotalLength = Math.max(totalLength, totalLength + child.getMeasuredHeight() +
                    lp.topMargin + lp.bottomMargin + getNextLocationOffset(child));
        }

        // Add in our padding
        mTotalLength += mPaddingTop + mPaddingBottom;
        // TODO: Should we recompute the heightSpec based on the new total length?
    } else {
        alternativeMaxWidth = Math.max(alternativeMaxWidth,
                                       weightedMaxWidth);


        // We have no limit, so make all weighted views as tall as the largest child.
        // Children will have already been measured once.
		//当子View已经计算后，不满足skippedMeasure = true 或者无剩余的空间
		//当子View使用最大宽度时，并且高度模式为wrap_content的时候
		//按照最大的高度值计算高度
        if (useLargestChild && heightMode != MeasureSpec.EXACTLY) {
            for (int i = 0; i < count; i++) {
                final View child = getVirtualChildAt(i);
                if (child == null || child.getVisibility() == View.GONE) {
                    continue;
                }

                final LinearLayout.LayoutParams lp =
                        (LinearLayout.LayoutParams) child.getLayoutParams();

                float childExtra = lp.weight;
                if (childExtra > 0) {
                    child.measure(
                            MeasureSpec.makeMeasureSpec(child.getMeasuredWidth(),
                                    MeasureSpec.EXACTLY),
                            MeasureSpec.makeMeasureSpec(largestChildHeight,
                                    MeasureSpec.EXACTLY));
                }
            }
        }
    }

    if (!allFillParent && widthMode != MeasureSpec.EXACTLY) {
        maxWidth = alternativeMaxWidth;
    }
    
    maxWidth += mPaddingLeft + mPaddingRight;

    // Check against our minimum width
    maxWidth = Math.max(maxWidth, getSuggestedMinimumWidth());
    
    setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
            heightSizeAndState);

    if (matchWidth) {
        forceUniformWidth(count, heightMeasureSpec);
    }
    }

在第三次测量的过程中，对那些跳过前面测量的控件进行测量，即对那些设置了weight属性的控件进行测量，这部分也是LinearLayout测量过程中重要的组成部分，从上面的分析来看，在进行weight的测量过程中，首先计算出LinearLayout还剩余的分配控件，接着判断当前View是否是前面跳过测量的view，如果是则进行子View的遍历，获取子view的LayoutParams属性中的weight值，用来计算该子View所分配的空间。接着测量view的高度，同时进行高度的设置。

通过对LinearLayout的onMeasure方法的通篇解读，我们可以得到以下结论：

**一、LinearLayout的高度计算是有针对性的**<br>
在测量1的工程中，我们可以发现当满足一定条件时，子view的高度就不会被计算。

* 当View的visiable属性设置为View.GONE时，此时在LinearLayout的高度测量中就会忽略这个view。
