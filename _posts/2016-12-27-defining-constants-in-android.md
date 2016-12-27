---
layout: post
title: Defining Constants in Android
description: We Android Developers often need to define constants to achieve better readability compared to just hard-coding some value into our code. In Android we have three choices to define constants
---

We Android Developers often need to define constants to achieve better readability compared to just hard-coding some value into our code. In Android we have three choices to define constants :  

- final static variable 
- Enum
- @IntDef or @StringDef

Every choice have their own advantage and disadvantage, Let's take a closer look at each of these choices.

## final static variable

This choice is the easiest and maybe the most used way to define constants.

```
class ComplaintReport {
    public final static int COMPLAINT_REASON_DEFAMATION = 0;
    public final static int COMPLAINT_REASON_PRIVACY_VIOLATION = 1;
    public final static int COMPLAINT_REASON_TROLL = 2;
}
```

Declaring constants this way is easier because it can be added to existing class and it will occupy less space compared to Enum, but if you declare your constants this way you won't get any type safety, for example :

```
class ComplaintReport {
    public final static int COMPLAINT_REASON_DEFAMATION = 0;
    public final static int COMPLAINT_REASON_PRIVACY_VIOLATION = 1;
    public final static int COMPLAINT_REASON_TROLL = 2;
    
    int complaintReason;
    
    public void setComplaintReason(int complaintReason) {
    	this.complaintReason = complaintReason;
    }
}
```

You can see that we can pass any `int` into `setComplaintReason(int)` and the compiler won't give us any warning because it's correct for now, but it can also lead us to unexpected behaviour when the passed value is incorrect like this :

```
public func foo() {
	ComplaintReport complaintReport = new ComplaintReport();
	complaintReport.setComplaintReason(3); 
}
```
The code above will compile, but there will be a potential bug because we are not expecting that value to be passed into our function (need to check bad value) and also there are nothing preventing a collision of constants, we can made a mistake and set two constants with same value.

## Enum

Declaring constants using Enum provide us type-safety and a lot of flexibility.  
Let's address the type-safety problem we have above, with enum our code will become like this :


```
public enum ComplaintReason {
	DEFAMATION, PRIVACY_VIOLATION, TROLL
}

class ComplaintReport {
    
    private ComplaintReason complaintReason;
    
    public void setComplaintReason(ComplaintReason complaintReason) {
    	this.complaintReason = complaintReason;
    }
}
```
So with changing our plain int to ComplaintReason, now compiler can know if we pass wrong value to `setComplaintReason(ComplaintReason)` which will be better because the error will occurs at compile time rather than runtime.   With enum our problem with collision of constants also won't occur because Enum values are unique.

We also get a lot of flexibilty with enum because we can define our own function inside enum and we can do method overloading too.

```
public enum Fish {
	TUNA {
		@Override
		public String getGenusName() {
			return "Thunnus";
		}
		
		@Override
		public String getFamilyName() {
			return "Scombridae";
		}
	}
	PINK_SALMON {
		@Override
		public String getGenusName() {
			return "Oncorhynchus";
		}
		
		@Override
		public String getFamilyName() {
			return "Salmonidae";
		}
	}
	LARGEMOUTH_BASS {
		@Override
		public String getGenusName() {
			return "Micropterus";
		}
		
		@Override
		public String getFamilyName() {
			return "Centrarchidae";
		}
	}
	
	public abstract String getGenusName()
	public abstract String getFamilyName()
	
}
```
As you can see, this kind of flexibilty can't be achieved with just a simple `final static variable` constants.

Some people claim that we can use Enum in a bitfield like this :

```
int selectedComplaintReasons = ComplaintReport.COMPLAINT_REASON_DEFAMATION | ComplaintReport.COMPLAINT_REASON_TROLL;
```

But actually we can use [EnumSet](https://developer.android.com/reference/java/util/EnumSet.html)

```
Set<ComplaintReason> selectedComplaintReasons = EnumSet.of(ComplaintReason.DEFAMATION, ComplaintReason.TROLL);
```

As always, with every advantages there will always disadvantages, enums often require more memory compared to simple `final static variable` constants, but it doesn't mean we shouldn't use enum at all.

## @IntDef and @StringDef

Google introduced [@IntDef](https://developer.android.com/reference/android/support/annotation/IntDef.html) and [@StringDef](https://developer.android.com/reference/android/support/annotation/StringDef.html) into their [Android Support Annotation](https://developer.android.com/reference/android/support/annotation/package-summary.html), because they want provide another way to define constants that use small memory while maintaining type-safety too.  
Let's take a look at our code with @IntDef :

```
class ComplaintReport {
    public final static int COMPLAINT_REASON_DEFAMATION = 0;
    public final static int COMPLAINT_REASON_PRIVACY_VIOLATION = 1;
    public final static int COMPLAINT_REASON_TROLL = 2;
    
	@IntDef({
	  COMPLAINT_REASON_DEFAMATION,
	  COMPLAINT_REASON_PRIVACY_VIOLATION,
	  COMPLAINT_REASON_TROLL
	})
    @Retention(RetentionPolicy.SOURCE)
	public @interface ComplaintReason { }

    @ComplaintReason int complaintReason;
    
    public void setComplaintReason(@ComplaintReason int complaintReason) {
    	this.complaintReason = complaintReason;
    }
}
```

You can see that the code is pretty similar with the first example with `final static variable` constants, but now we have @ComplaintReason in front of our variable and setter's parameter. With this we can get type-safety that we don't have before, we can't pass incorrect value to `setComplaintReason(@ComplaintReason int complaintReason)` anymore because the compiler will check if is the value passed is the value included inside @IntDef or not.  
So now this code won't compile :

```
public func foo() {
	ComplaintReport complaintReport = new ComplaintReport();
	complaintReport.setComplaintReason(3); // Will give you an error that told you to use one of the value inside @IntDef
}
```

Using @IntDef will also have same memory size as `final static variable` constants because everything is done in compile time, as you can see we define it with `@Retention(RetentionPolicy.SOURCE)` which means that the annotated type will be discarded by the compiler, you can take a look at other [RetentionPolicy](https://docs.oracle.com/javase/7/docs/api/java/lang/annotation/RetentionPolicy.html) if you are interested in it.

## Ending Words

Like I said before every choices have their own advantages and disadvantages, usually I choose between Enum or @IntDef/@StringDef based on the use-case.
For example if I have just want my constants to be simple values, I will use @IntDef/@StringDef. If I want it to be more complex, I'll use Enums even it consumes more memory it provide flexibility like I explained above. 

Thanks for reading and if you have any question you can write comment below or reach me via twitter [@niko_yuwono](https://twitter.com/niko_yuwono).
