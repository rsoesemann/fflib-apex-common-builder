Test data Builder for FinancialForce Apex Common
================================================
*fflib-apex-common-builder* is an extension to [ApexCommon](https://github.com/financialforcedev/fflib-apex-common) and helps you write better [integration tests](https://en.wikipedia.org/wiki/Integration_testing). Better in the sense of shorter, more concise. And by that more readable, more understandble.
    
[![Build Status](https://travis-ci.org/financialforcedev/fflib-apex-common-builder.svg)](https://travis-ci.org/financialforcedev/fflib-apex-common-builder) 

**Dependencies:** Must deploy [ApexCommon](https://github.com/financialforcedev/fflib-apex-common) before deploying this library

<a href="https://githubsfdeploy.herokuapp.com?owner=financialforcedev&repo=fflib-apex-common">
  <img alt="Deploy to Salesforce"
       src="https://raw.githubusercontent.com/afawcett/githubsfdeploy/master/src/main/webapp/resources/img/deploy.png">
</a>

See here for [MavensMate Templates](https://github.com/joeferraro/MavensMate-Templates/pull/18/files) for generating fflib_DomainObjectBuilder Base & Domain specific classes.

History
=======
- **March 2017**, Accepted and merged by [@afawcett](https://github.com/afawcett) into the Financial Force Github org as an extension project of Apex Commons
- **January 2016**, Huge feature extension (In-memory persistence, selectice persistence, Object Mother pattern) by [@jondavis9898](https://github.com/jondavis9898) in https://github.com/financialforcedev/fflib-apex-common/pull/100 
- **August 2015**, Initial contribution by [@up2go-rsoesemann](https://github.com/up2go-rsoesemann) of a Test Builder base class in  https://github.com/financialforcedev/fflib-apex-common/pull/77.

About this library
==================
- Q: Why do we need this library?  
   - *A: Because you need integration tests.*
- Q: I have integration tests!   
   - *A: Sure. We all do. But they are complex and by that hard read and maintain.*
- Q: Why that?   
   - *A: Because 90% of the LOC are setup code. And the majority cares about creating records in the right order.*
- Q: Ok, but isn't that how Salesforce works.   
   - *A: Maybe. But why not make our lifes easiert and hide all the technical plumbing away?*
   
### Integration test are crucial   

We need integration test for every non-trivial Salesforce application, simply because they are database applications. If we don't prove that all those well-tested, perfectly isolated or mocked units work with the database we don't know that our application will work in the real world.

### Good test = Reveals its intention + Fits one Screen

Writing Apex integration tests is a pain. Just take this integration test from a [Trailhead module](https://trailhead.salesforce.com/modules/apex_patterns_sl/units/apex_patterns_sl_apply_uow_principles)

    @isTest
    private static void testService() {
    
        // Setup
        Opportunity opp = new Opportunity();
        opp.Name = 'Opportunity ' + o;
        opp.StageName = 'Open';
        opp.CloseDate = System.today();

        List<Product2> products = new List<Product2>();
        List<PricebookEntry> pricebookEntries = new List<PricebookEntry>();
        List<OpportunityLineItem> oppLineItems = new List<OpportunityLineItem>();
        
        for(Integer i=0; i<5; i++) {                       
            Product2 product = new Product2();
            product.Name = opp.Name + ' : Product : ' + i;
            products.add(product);

            PricebookEntry pbe = new PricebookEntry();
            pbe.UnitPrice = 10;
            pbe.IsActive = true;
            pbe.UseStandardPrice = false;
            pbe.Pricebook2Id = Test.getStandardPricebookId();
            pricebookEntries.add(pbe);

            OpportunityLineItem oppLineItem = new OpportunityLineItem();
            oppLineItem.Quantity = 1;
            oppLineItem.TotalPrice = 10;
            oppLineItems.add(oppLineItem);
        }
        
        insert opp;



        // Exercise
        List<Id> invoiceIds = InvoicingService.generate(new List<Id> { opp.Id });


        // Verify
        System.assertEquals(1, invoiceIds.size());
    }

Leveraging the awesome [fflib_ISObjectUnitOfWork from Apex Common](https://github.com/financialforcedev/fflib-apex-common/blob/master/fflib/src/classes/fflib_SObjectUnitOfWork.cls) takes away the complexity related to relationships and the order in which you have to insert objects. But having all the technical aspects of the Unit of Work spreaded in the setup code is still far from optimal.

    @isTest
    private static void testService_UsingUnitOfWork() {
    
        // Setup
        fflib_ISObjectUnitOfWork uow = Application.UnitOfWork.newInstance();
        
        Opportunity opp = new Opportunity();
        opp.Name = 'Opportunity';
        opp.StageName = 'Open';
        opp.CloseDate = System.today();
        uow.registerNew(opp);     
        
        for(Integer i=0; i<5; i++) {                       
            Product2 product = new Product2();
            product.Name = opp.Name + ' : Product : ' + i;
            uow.registerNew(product);  
            
            PricebookEntry pbe = new PricebookEntry();
            pbe.UnitPrice = 10;
            pbe.IsActive = true;
            pbe.UseStandardPrice = false;
            pbe.Pricebook2Id = Test.getStandardPricebookId();
            uow.registerNew(pbe, PricebookEntry.Product2Id, product);       
            
            OpportunityLineItem oppLineItem = new OpportunityLineItem();
            oppLineItem.Quantity = 1;
            oppLineItem.TotalPrice = 10;
            
            uow.registerRelationship(oppLineItem, OpportunityLineItem.PricebookEntryId, pbe);
            uow.registerNew(oppLineItem, OpportunityLineItem.OpportunityId, opp);
        }
        
        uow.commitWork();


        // Exercise
        List<Id> invoiceIds = InvoicingService.generate(new List<Id> { opp.Id });


        // Verify
        System.assertEquals(1, invoiceIds.size());
    }
    
 20 lines of Setup compared to 2 lines for the rest. That's way to much. And most of those 20 line just configure the UOW or set record default fields. 
 
 Imagine the code could look like this.
 
     @isTest
     private static void testService_UsingTestDataBuilders() {
        
        // Setup
        Opportunity_t opp = new Opportunity_t()
                                    .stage('Open')
                                    .closes(System.today());
        
        for(Integer i=0; i<5; i++) {
          opp.add(new OpportunityLineItem_t(new PriceBookEntry_t(new Product_t('Product ' + i))
                                                 .price(10)
                                                 .useStdPrice(false))
                           .quantity(1)
                           .totalPrice(10));
        }
            
        opp.build(); 
    

        // Exercise
        List<Id> invoiceIds = InvoicingService.generate(new List<Id> { opp.record.Id });

        
        // Verify
        System.assertEquals(1, invoiceIds.size());
    }
 
