

# Example case : Modelling a busines state machine


## Introduction to the problem

We would like to model certain requirements on the execution of the program:

-   The flow of stages in the process will be modified by certain actions. Those actions will also only be available at a particular stage of the process.
-   A certain set of validations have to be conducted before the next stage of the process can be achieved


## Basic Business models

  

    data User

    data Product

    data Cart = MkCart [Product]

    data PaymentDetails


## Validations and process stages

  

    data Validation
      = AuthUser
      | ValidCart
      | ValidPaymentDetails
      | ConfirmedPurchase

  

    data CheckoutState
      = New
      | InProcess
      | Complete


## State aggregation model

  

    data CheckoutSummary (a :: [Validation]) (b :: CheckoutState)
      = MkSummary (User, Cart, PaymentDetails)


## Application Effect type

  

    type AppM = EitherT ClientError IO
    
    data ClientError
      = ConnectionTimeout
      | NotFound


## Validation functions

    
    authUser :: CheckoutSummary a b
              -> AppM (CheckoutSummary (a |+| 'AuthUser) b)

  

    validateCart :: CheckoutSummary a b
                  -> AppM (CheckoutSummary (a |+| 'ValidCart) b)

  

    checkPaymentDetails :: CheckoutSummary a b
                         -> AppM (CheckoutSummary (a |+| 'ValidPaymentDetails) b)

  

    confirmPurchase :: CheckoutSummary a b
                     -> AppM (CheckoutSummary (a |+| 'ConfirmedPurchase) b)


## Stage Update functions

  

    checkout :: Must (ContainsMany a ['AuthUser, 'ValidPaymentDetails])
              => CheckoutSummary a 'New
              -> AppM (CheckoutSummary a 'InProcess)

  

    completePurchase :: Must (ContainsMany a ['AuthUser, 'ValidCart,
                                              'ValidPaymentDetails,
                                              'ConfirmedPurchase])
                     => CheckoutSummary a 'InProcess
                     -> AppM (CheckoutSummary a 'Complete)


## TypeLevel helper functions

  

    type family (xs :: [a]) |+| (y :: a) :: [a]

  

    type family ContainsMany (xs :: [t]) (ys :: [t]) :: Bool

  

    type family Must (x :: Bool) :: Constraint

