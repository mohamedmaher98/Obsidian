
| رقم الخطأ  | وصف المشكلة                                                                                                                               |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SRDRQ05725 | ان لما بختار بناء علي في حقل اسمه returnDate مش متعرف في الكلاسات الي موجوده في بناء علي                                                  |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ECDR06791  | لو عملت جرد مخزني وجيت اعمل عملية بيع علي POS بيطلع ايرور عادي بس بيحفظها<br>انا عايز لما يحصل كده يطلع الايرور بس مايحفظهاش في نما درافت |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
| ECDR06750  | ماينفعش اعمل اذنين انصراف الوقت بتاعهم متداخل في  نفس اليوم لنفس الموظف                                                                   |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
|            |                                                                                                                                           |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
|            |                                                                                                                                           |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
|            |                                                                                                                                           |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
|            |                                                                                                                                           |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
|            |                                                                                                                                           |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
|            |                                                                                                                                           |     |     |     |     |     |     |     |     |     |     |     |     |     |     |
|            |                                                                                                                                           |     |     |     |     |     |     |     |     |     |     |     |     |     |     |

### ==**ECDR06750**==

isValidForCommit()
validateNoOverlappingPermissionInTheSameDay(result);

```java
private void validateNoOverlappingPermissionInTheSameDay(Result result)  
{  
    Criteria criteria = CriteriaBuilder.create().field(IdsOfLeavePermission.id).notEqual(getId()) // هات كل الاذونات الي الي دي بتاعها مش انا 
    .and().field(IdsOfLeavePermission.employee).equal(getEmployee()).and()  //بفلتر عشان اجيب الاذونات لنفس الموظف
          .areDatesOverLapping(IdsOfLeavePermission.fromDate, IdsOfLeavePermission.toDate, getFromDate(), getToDate()).and()//مايكونش في تداخل فالتاريخ
          
          .commitedBefore()  //يكون اتحفظ قبل كده مش درافت 
          .build();  
    List<LeavePermission> existingLeavePermission = Persister.listPageMatching(LeavePermission.class, PageData.All(), "", criteria,  
          FilterOnDimensions.No);  
    for (LeavePermission leavePermission : existingLeavePermission)  
    {  
       if (TimeDF.areTimesOverlapping(leavePermission.getFromHour(), leavePermission.getToHour(), getFromHour(), getToHour()))  
          Result.createFailureResult("Leave permission {0} overlaps with {1} for the employee {2}, From Date {3}", leavePermission, this,  
                getEmployee(), getFromDate()).relatedToRecord(leavePermission).alwaysShowRelatedTo().addToAccumulatingResult(result);  
    }  
}

public static boolean areTimesOverlapping(TimeDF ts1, TimeDF te1, TimeDF ts2, TimeDF te2)  
{  
    Integer s1 = ts1 == null ? null : ts1.getPrimitiveValue();  
    Integer e1 = te1 == null ? null : te1.getPrimitiveValue();  
    Integer s2 = ts2 == null ? null : ts2.getPrimitiveValue();  
    Integer e2 = te2 == null ? null : te2.getPrimitiveValue();  
    if (e1 == null)  
       e1 = Integer.MAX_VALUE;  
    if (e2 == null)  
       e2 = Integer.MAX_VALUE;  
    if (s1 == null)  
       s1 = Integer.MIN_VALUE;  
    if (s2 == null)  
       s2 = Integer.MIN_VALUE;  
    return e1 >= s2 && e2 >= s1;  
}
```
