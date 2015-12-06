Checks
======

`CHECK` forbids records to have values that do not meet some conditions.

We know that Apple devices have push tokens of 32 symbols (device_vendor is a enum):

    ALTER TABLE public.users_push_tokens
        ADD CONSTRAINT users_push_token_length_check
        CHECK  (CASE WHEN vendor == 'apple'::device_vendor THEN length(token) = 32
                     WHEN vendor == 'google'::device_vendor THEN length(token) = 140
                     ELSE TRUE
                END);

So

    INSERT INTO public.users_push_tokens
        (vendor, push_token)
        VALUES (
            'apple',
            'WRONG_TOKEN'
        );

will throw an exception.

`CHECK` can not be deferred (as FK).
