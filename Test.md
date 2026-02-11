                                                                                                                                                                                                                                             
● ملخص التعديلات في الـ Change List
                                                                                                                                                                                                                                             
  التعديلات بتتقسم لـ 3 أجزاء رئيسية:                                                                                                                                                                                                        
    
  ---                                                                                                                                                                                                                                        
  1. إضافة Entity جديد: REMaintenanceFund (صندوق الصيانة)                       

  تم إنشاء entity جديد بالكامل في موديول الـ Real Estate بكل الطبقات:

  - DSL - تعريف الـ entity بالحقول: project, totalProjectare, bank, bankAccount, totalMaintenanceDeposit, totalCollectedMaintenanAmount, remainingBalance
  - Domain - الـ Generated entity + DataHolder + الـ entity class
  - Contracts - الـ DTO + Generated DTO + Web Service proxy
  - Services - الـ Factory + WS Wrapper + WS Implementation
  - تسجيل الـ Entity في REEntities, REEntityTypes, RENormalBasicFactoryProvider

  وكمان تم إضافة حقل maintenanceFund في RERentalUnit (الوحدة الإيجارية) عشان يربطها بصندوق الصيانة.

  ---
  2. تعديل validation البنود المكررة في الـ Contracting

  تعديل في 3 ملفات:
  - AbstractContract - تغيير منطق التحقق من تكرار الـ Term Code بحيث يسمح بتكرار الـ Term Code لو الـ StandardTerm نفسه، ويمنع التكرار فقط لو الـ StandardTerm مختلف
  - ContractingCostExecution - نفس التعديل في تنفيذ التكاليف
  - ContractsUtil - نفس التعديل في الـ utility class

  ---
  3. تعديلات بنية وإعدادات

  - modstorun/pom.xml - تفعيل كل الموديولات (كانت مُعلّق عليها بـ comments) وإضافة موديولات جديدة: magento, namapos, realestate, hms, frm, travel, education
  - module-info.java - تعطيل export الـ userfeatures package
  - nama.properties - تغيير إعدادات الاتصال (server URL, DB password, login credentials, registry code)
