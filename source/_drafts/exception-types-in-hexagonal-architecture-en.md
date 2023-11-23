exceptions types:
- Domain:
extend from: Shared\Domain\Exceptions\DomainException
thrown from: Domain
handled in: propagate unhandled (causing 400 or 422 HTTP response codes), or, optionally, are converted to Application ones in Application
description: mean that domain object received broken data; could (and should) be prevented by better input validation
examples: ChargeInvalidAmountException, InvoiceCannotBePaidException, etc.

	- Application:
			extend from: Shared\Application\Exceptions\ApplicationException
			thrown from: Application
			handled in:
				type 1: in controllers
				type 2: do not imply handling, cause 404 or 409 HTTP response codes
			description: 
				type 1: used only for flow control, not meaning that something is really broken, should be replaced with Result objects in the future 
				type 2: all the "entity not found" and "entity already exists" exceptions
			examples: 
				type 1: ThreeDsRequiredException, RedirectRequiredException, etc
				type 2: ChargeNotFoundException, EmailAlreadyInUseException, etc

	- Infrastructure:
			extend from: Shared\Infrastructure\Exceptions\InfrastructureException
			thrown from: Infrastructure
			handled in: propagate unhandled (causing 500 HTTP response codes), or, optionally, are converted to Application ones in Application
			description: something went wrong on the lowest level
			examples: ThirdPartyApiUnavailableException, DatabaseDuplicateEntryException, etc