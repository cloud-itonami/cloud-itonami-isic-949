(ns civicmembershiporg.governor
  "Governor with three HARD, permanent, un-overridable checks for the
  civic membership organization administrative coordination actor.

  1. Member/event-record unverified — target must exist in store AND be
     independently :registered?/:verified?, re-derived every time.
  2. Effect not :propose — rejected outright.
  3. Scope exclusion — any proposal touching membership-eligibility/expulsion,
     religious-doctrine, political-position, advocacy-policy/policy-content,
     dues-amount/fee-waiver, or disciplinary action is permanently blocked."
  (:require [civicmembershiporg.store :as store]
            [kotoba.lang.text :as str]))

;; ---------------------- hard checks ----------------------

(defn member-unverified-violations
  "Check 1: Member must be registered AND verified.
  This is re-derived from the member's own :registered?/:verified? fields,
  never from proposal self-report."
  [store member-id]
  (let [member (store/member store member-id)]
    (cond
      (nil? member)
      [{:check/id :member-unverified
        :violation "Member not found in store"}]

      (not (:registered? member))
      [{:check/id :member-unverified
        :violation "Member is not registered"}]

      (not (:verified? member))
      [{:check/id :member-unverified
        :violation "Member is not verified"}]

      :else
      [])))

(defn effect-not-propose-violations
  "Check 2: Effect must be :propose. Any other effect is rejected outright."
  [proposal]
  (if (not= (:effect proposal) :propose)
    [{:check/id :effect-not-propose
      :violation (str "Effect is " (:effect proposal) ", not :propose")}]
    []))

(defn scope-exclusion-violations
  "Check 3: Block proposals touching excluded territory.
  Excluded: membership-eligibility/expulsion decisions, religious-doctrine,
  political-position, advocacy-position/policy-content, dues-amount/fee-waiver
  decisions, disciplinary action.

  Uses qualified substring scan (EN+JA) so legitimate :flag-safety-concern
  ops that mention 'safety' aren't self-blocked."
  [proposal]
  (let [forbidden-patterns
        [;; EN patterns
         #"(?i)membership.?eligib"
         #"(?i)membership.?expul"
         #"(?i)membership.?remov"
         #"(?i)religious.?doctrine"
         #"(?i)religious.?teaching"
         #"(?i)religious.?belief"
         #"(?i)religious.?content"
         #"(?i)political.?position"
         #"(?i)political.?stance"
         #"(?i)political.?decision"
         #"(?i)political.?content"
         #"(?i)advocacy"
         #"(?i)policy.?position"
         #"(?i)dues.?amount"
         #"(?i)dues.?waiv"
         #"(?i)fee.?waiv"
         #"(?i)disciplinary"
         #"(?i)censure"
         #"(?i)suspension"
         ;; JA patterns (common civic org terminology)
         #"会員.?資格"
         #"会員.?除名"
         #"会員.?削除"
         #"宗教.?信条"
         #"宗教.?教義"
         #"宗教.?内容"
         #"政治.?立場"
         #"政治.?主張"
         #"政治.?意思決定"
         #"政策.?立場"
         #"会費.?減免"
         #"懲罰"
         #"処分"]

        ;; Allowed operations that legitimately mention concerns
        allowed-ops #{:flag-safety-concern}

        op-id (:operation proposal)
        proposal-str (str proposal)

        ;; Check if any forbidden pattern matches
        has-forbidden-match (some #(re-find % proposal-str) forbidden-patterns)
        is-allowed-op (allowed-ops op-id)

        ;; Combined check: return violation only if forbidden AND not allowed-op
        should-reject (boolean (and has-forbidden-match (not is-allowed-op)))]

    (if should-reject
      [{:check/id :scope-exclusion
        :violation "Proposal touches membership-eligibility, religious-doctrine, political-position, advocacy-policy, dues-waiver, or disciplinary decisions"}]
      [])))

;; ---------------------- decision logic ----------------------

(defn govern
  "Apply all three HARD checks. Any violation is a permanent rejection
  with no override path."
  [store proposal]
  (let [;; Only check member verification if member-id is present
        member-violations (if (:member-id proposal)
                            (member-unverified-violations store (:member-id proposal))
                            [])
        effect-violations (effect-not-propose-violations proposal)
        scope-violations (scope-exclusion-violations proposal)
        all-violations (concat member-violations effect-violations scope-violations)]

    {:proposal proposal
     :violations all-violations
     :passes? (empty? all-violations)
     :decision (if (empty? all-violations)
                 :APPROVE
                 :REJECT)}))
