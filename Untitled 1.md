```java
public void processNumbers(List<Integer> nums, Predicate<Integer> p) {
    for (int n : nums) {
        if (p.test(n)) // لو الشرط true
            System.out.println(n);
    }
}





List<Integer> numbers = List.of(1, 2, 3, 4, 5);


public list<invoice> filterInvoices(List<invoice> invoices , predecate<invoice> operatin)
{
	List<invoice> newInvoices = new ArrayList<>;
	for(invoice i : invoices)
	{
		if (operation.test(i))
			newInvoices.add(i);
	}
	return newInvoice();
} 


processNumbers(numbers , s -> System.out.println("Number"+s.toString()))





























```
اكتب method بتاخد الـ list دي وتحول كل رقم لـ String على الشكل ده: