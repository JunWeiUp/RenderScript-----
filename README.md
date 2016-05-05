# RenderScript- 模糊算法

	
Fragment
	
	package net.qiujuer.sample.blur.fragments;
	
	import android.annotation.TargetApi;
	import android.graphics.Bitmap;
	import android.graphics.Canvas;
	import android.graphics.Paint;
	import android.graphics.drawable.BitmapDrawable;
	import android.os.Build;
	import android.os.Bundle;
	import android.renderscript.Allocation;
	import android.renderscript.RenderScript;
	import android.renderscript.ScriptIntrinsicBlur;
	import android.support.v4.app.Fragment;
	import android.view.LayoutInflater;
	import android.view.View;
	import android.view.ViewGroup;
	import android.view.ViewTreeObserver;
	import android.widget.CheckBox;
	import android.widget.ImageView;
	import android.widget.TextView;
	
	import net.qiujuer.sample.blur.R;
	
	
	/**
	 * Created by jin
	 * on 2014/4/19.
	 */
	public class RSBlurFragment extends Fragment {
	private final String DOWNSCALE_FILTER = "downscale_filter";
	    private ImageView image;
	    private TextView text;
	    private TextView statusText;
	    private CheckBox downScale;
	
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
	        View view = inflater.inflate(R.layout.fragment_layout, container, false);
	image = (ImageView) view.findViewById(R.id.picture);
	text = (TextView) view.findViewById(R.id.text);
	image.setImageResource(R.drawable.picture);
	        if (savedInstanceState != null) {
	downScale.setChecked(savedInstanceState.getBoolean(DOWNSCALE_FILTER));
	}
	        applyBlur();
	        return view;
	}
	
	private void applyBlur() {
	image.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
	@Override
	public boolean onPreDraw() {
	image.getViewTreeObserver().removeOnPreDrawListener(this);
	image.buildDrawingCache();
	
	Bitmap bmp = image.getDrawingCache();
	blur(bmp, text);
	                return true;
	}
	        });
	}
	
	@TargetApi(Build.VERSION_CODES.KITKAT)
	private void blur(Bitmap bkg, View view) {
	
	float scaleFactor = 1;
	        float radius = 25;
	
	Bitmap overlay = Bitmap.createBitmap((int) (view.getMeasuredWidth() / scaleFactor),
	(int) (view.getMeasuredHeight() / scaleFactor), Bitmap.Config.ARGB_8888);
	
	Canvas canvas = new Canvas(overlay);
	
	canvas.translate(-view.getLeft() / scaleFactor, -view.getTop() / scaleFactor);
	canvas.scale(1 / scaleFactor, 1 / scaleFactor);
	Paint paint = new Paint();
	paint.setFlags(Paint.FILTER_BITMAP_FLAG);
	canvas.drawBitmap(bkg, 0, 0, paint);
	
	RenderScript rs = RenderScript.create(getActivity());
	
	Allocation overlayAlloc = Allocation.createFromBitmap(
	                rs, overlay);
	
	ScriptIntrinsicBlur blur = ScriptIntrinsicBlur.create(
	                rs, overlayAlloc.getElement());
	
	blur.setInput(overlayAlloc);
	
	blur.setRadius(radius);
	
	blur.forEach(overlayAlloc);
	
	overlayAlloc.copyTo(overlay);
	
	text.setBackground(new BitmapDrawable(
	                getResources(), overlay));
	
	rs.destroy();
	}
	
	@Override
	public String toString() {
	return "RenderScript";
	}
	
	
	}

xml
	<?xml version="1.0" encoding="utf-8"?>
	
	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="match_parent"
	android:layout_height="match_parent">
	
	    <ImageView
	android:id="@+id/picture"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:scaleType="centerCrop" />
	
	    <TextView
	android:id="@+id/text"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:layout_gravity="center_vertical"
	android:gravity="center_horizontal|center_vertical"
	android:text="Blur Text"
	android:textColor="@android:color/white"
	android:textSize="60sp"
	android:textStyle="bold" />
	
	    <LinearLayout
	android:id="@+id/controls"
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:layout_gravity="bottom"
	android:background="#7fffffff"
	android:orientation="vertical" />
	</FrameLayout>