---
title: "Android 14/15のTimePickerがアプリをクラッシュさせることがある"
emoji: "💣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["android"]
published: true
publication_name: "aldagram_tech"
---

[`TimePicker`](https://developer.android.com/reference/android/widget/TimePicker) や [`TimePickerDialog`](https://developer.android.com/reference/android/app/TimePickerDialog) で時間を変更しようとすると例外が発生してアプリをクラッシュさせるといった事象が報告されていたようです。

https://issuetracker.google.com/issues/333670354

- 発生条件
    - Android 14もしくは15の一部機種
    - スピナーモードにしている
    - AM/PMの選択がボタンになるスタイル（`Theme.AppCompat.Light` など）を当てている
- ログ

```
java.lang.NullPointerException: Attempt to invoke virtual method 'boolean android.widget.EditText.hasFocus()' on a null object reference
  at android.widget.TimePickerSpinnerDelegate.updateInputState(TimePickerSpinnerDelegate.java:480)
  at android.widget.TimePickerSpinnerDelegate.-$$Nest$mupdateInputState(Unknown Source:0)
  at android.widget.TimePickerSpinnerDelegate$2.onValueChange(TimePickerSpinnerDelegate.java:119)
  at android.widget.NumberPicker.notifyChange(NumberPicker.java:2080)
  at android.widget.NumberPicker.setValueInternal(NumberPicker.java:1850)
  at android.widget.NumberPicker.scrollBy(NumberPicker.java:1189)
  at android.widget.NumberPicker.onTouchEvent(NumberPicker.java:969)
  ...
```

## 原因

ログにある `EditText.hasFocus()` は `TimePickerSpinnerDelegate.updateInputState()` で呼ばれています。

```java
// https://android.googlesource.com/platform/frameworks/base/+/2a757ef4aea5cf9674be508e87378f66874ef163/core/java/android/widget/TimePickerSpinnerDelegate.java#466
    private void updateInputState() {
        // Make sure that if the user changes the value and the IME is active
        // for one of the inputs if this widget, the IME is closed. If the user
        // changed the value via the IME and there is a next input the IME will
        // be shown, otherwise the user chose another means of changing the
        // value and having the IME up makes no sense.
        InputMethodManager inputMethodManager = mContext.getSystemService(InputMethodManager.class);
        if (inputMethodManager != null) {
            if (mHourSpinnerInput.hasFocus()) {
                inputMethodManager.hideSoftInputFromView(mHourSpinnerInput, 0);
                mHourSpinnerInput.clearFocus();
            } else if (mMinuteSpinnerInput.hasFocus()) {
                inputMethodManager.hideSoftInputFromView(mMinuteSpinnerInput, 0);
                mMinuteSpinnerInput.clearFocus();
            // 👇
            } else if (mAmPmSpinnerInput.hasFocus()) {
                inputMethodManager.hideSoftInputFromView(mAmPmSpinnerInput, 0);
                mAmPmSpinnerInput.clearFocus();
            }
        }
    }
```

この `mAmPmSpinnerInput` は `TimePickerSpinnerDelegate.TimePickerSpinnerDelegate()` で初期化されています。

```java
// https://android.googlesource.com/platform/frameworks/base/+/2a757ef4aea5cf9674be508e87378f66874ef163/core/java/android/widget/TimePickerSpinnerDelegate.java#146
        // am/pm
        final View amPmView = mDelegator.findViewById(R.id.amPm);
        if (amPmView instanceof Button) {
            mAmPmSpinner = null;
            // 👇
            mAmPmSpinnerInput = null;
            mAmPmButton = (Button) amPmView;
            mAmPmButton.setOnClickListener(new View.OnClickListener() {
                public void onClick(View button) {
                    button.requestFocus();
                    mIsAm = !mIsAm;
                    updateAmPmControl();
                    onTimeChanged();
                }
            });
        } else {
            mAmPmButton = null;
            mAmPmSpinner = (NumberPicker) amPmView;
            mAmPmSpinner.setMinValue(0);
            mAmPmSpinner.setMaxValue(1);
            mAmPmSpinner.setDisplayedValues(mAmPmStrings);
            mAmPmSpinner.setOnValueChangedListener(new NumberPicker.OnValueChangeListener() {
                public void onValueChange(NumberPicker picker, int oldVal, int newVal) {
                    updateInputState();
                    picker.requestFocus();
                    mIsAm = !mIsAm;
                    updateAmPmControl();
                    onTimeChanged();
                }
            });
            // 👇
            mAmPmSpinnerInput = mAmPmSpinner.findViewById(R.id.numberpicker_input);
            mAmPmSpinnerInput.setImeOptions(EditorInfo.IME_ACTION_DONE);
        }
```

AM/PMを選択させる `amPmView` がボタン以外のときは `EditText` のインスタンスが設定されますが、ボタンのときは `null` のままなので `mAmPmSpinnerInput.hasFocus()` が例外を投げてしまうようですね。

## 対策

対応として確実なのはモードやスタイルを変えてしまうことですが、何らかの事情によりそれができない場合はスピナーに登録されているイベントリスナーを上書きするしかなさそうです。

```kotlin
class WorkaroundTimePicker(
  context: Context,
  attrs: AttributeSet,
) : TimePicker(context, attrs) {
  private val requiresWorkaroundForAndroid14And15: Boolean
    get() = (Build.VERSION.SDK_INT == Build.VERSION_CODES.UPSIDE_DOWN_CAKE || Build.VERSION.SDK_INT == Build.VERSION_CODES.VANILLA_ICE_CREAM)
      && findViewById<View?>(Resources.getSystem().getIdentifier("amPm", "id", "android")) is Button

  init {
    if (requiresWorkaroundForAndroid14And15) {
      val hourSpinner = findViewById<NumberPicker?>(Resources.getSystem().getIdentifier("hour", "id", "android"))
      val minuteSpinner = findViewById<NumberPicker?>(Resources.getSystem().getIdentifier("minute", "id", "android"))

      hourSpinner?.setOnValueChangedListener { spinner, oldVal, newVal ->
        ...
      }

      minuteSpinner?.setOnValueChangedListener { spinner, oldVal, newVal ->
        ...
      }
    }
  }
}
```

具体的に何を書くかは既存の実装が参考になるでしょう。

https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/widget/TimePickerSpinnerDelegate.java#87

以上です。

## 参考

https://android.googlesource.com/platform/frameworks/base/+/2a757ef4aea5cf9674be508e87378f66874ef163/core/java/android/widget/TimePickerSpinnerDelegate.java

https://qiita.com/t-kashima/items/9462af782fb5f1a2a7da
